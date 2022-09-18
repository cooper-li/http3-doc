## 摘要

The QUIC transport protocol has several features that are desirable in a transport for HTTP, such as stream multiplexing, per-stream flow control, and low-latency connection establishment. This document describes a mapping of HTTP semantics over QUIC. This document also identifies HTTP/2 features that are subsumed by QUIC and describes how HTTP/2 extensions can be ported to HTTP/3.

QUIC传输协议具有一些在HTTP传输中所需要的功能, 例如流多路复用, 每个流的流量控制和低延迟连接的建立。本文档描述了QUIC对于HTTP的语意映射关系。并介绍了QUIC包含的HTTP/2功能, 以及如何将HTTP/2扩展移植到HTTP/3。

## 此文档状态

This is an Internet Standards Track document.

This document is a product of the Internet Engineering Task Force (IETF). It represents the consensus of the IETF community. It has received public review and has been approved for publication by the Internet Engineering Steering Group (IESG). Further information on Internet Standards is available in Section 2 of RFC 7841.

Information about the current status of this document, any errata, and how to provide feedback on it may be obtained at <https://www.rfc-editor.org/info/rfc9114>.

这是一份互联网标准跟踪文档。

本文档是互联网工程任务组(IETF)的产品。它代表了IETF社区的共识。它已经接受了公众的审查，并已被互联网工程指导小组(IESG)批准发表。有关Internet标准的更多信息，请参阅RFC 7841的第2节。

有关本文档的当前状态、任何勘误表以及如何提供反馈的信息，请访问https://www.rfc-editor.org/info/rfc9114.

## 版权声明

Copyright (c) 2022 IETF Trust and the persons identified as the document authors. All rights reserved.

This document is subject to BCP 78 and the IETF Trust's Legal Provisions Relating to IETF Documents (<https://trustee.ietf.org/license-info>) in effect on the date of publication of this document. Please review these documents carefully, as they describe your rights and restrictions with respect to this document. Code Components extracted from this document must include Revised BSD License text as described in Section 4.e of the Trust Legal Provisions and are provided without warranty as described in the Revised BSD License.

版权所有(C)2022年IETF信托基金和被确认为文档作者的人。版权所有。

本文件受BCP78和IETF Trust有关IETF文件的法律规定(https://trustee.ietf.org/license-info))的约束，自本文件发布之日起生效。请仔细阅读这些文档，因为它们描述了您对本文档的权利和限制。本文档中提取的代码组件必须包括修订的BSD许可证文本，如信托法律条款的第4.e节所述，并且不提供修订的BSD许可证中所述的担保。

## 1. 引言

HTTP semantics ([[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)]) are used for a broad range of services on the Internet. These semantics have most commonly been used with HTTP/1.1 and HTTP/2. HTTP/1.1 has been used over a variety of transport and session layers, while HTTP/2 has been used primarily with TLS over TCP. HTTP/3 supports the same semantics over a new transport protocol: QUIC.

Internet上的服务广泛使用HTTP语义([[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)])。这些语义通常用来描述HTTP/1.1和HTTP/2。HTTP/1.1在各种运输层和会话层中都有使用, HTTP/2主要与基于TCP上层的TLS一起使用。HTTP/3基于一个新的传输协议: QUIC 也具有相同的语意。

### 1.1 HTTP早期版本

HTTP/1.1 ([[HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9112)]) uses whitespace-delimited text fields to convey HTTP messages. While these exchanges are human readable, using whitespace for message formatting leads to parsing complexity and excessive tolerance of variant behavior.

HTTP/1.1 ([[HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9112)]) 使用空格分隔的文本字段进行HTTP消息的传输。虽然有较强的的数据可读性, 但使用空格进行消息格式化会提高解析的复杂度和对不同行为的过度妥协。

Because HTTP/1.1 does not include a multiplexing layer, multiple TCP connections are often used to service requests in parallel. However, that has a negative impact on congestion control and network efficiency, since TCP does not share congestion control across multiple connections.

由于HTTP/1.1不包括多路复用层的定义, 因此通常浏览器通常会建立多个TCP连接并行的发起请求。然而这样对拥塞控制和网络效率或产生负面影响, 因为TCP不支持跨多个连接共享拥塞控制机制。

HTTP/2 ([[HTTP/2](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9113)]) introduced a binary framing and multiplexing layer to improve latency without modifying the transport layer. However, because the parallel nature of HTTP/2's multiplexing is not visible to TCP's loss recovery mechanisms, a lost or reordered packet causes all active transactions to experience a stall regardless of whether that transaction was directly impacted by the lost packet.

HTTP/2 ([[HTTP/2](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9113)]) 引入了二进制帧和多路复用机制, 可以在不修改传输层的情况下改善延迟。但是HTTP/2多路复用的并行特性对于TCP的丢包重传机制是不可见的, 因此不管活动中的事务(连接)是否受丢包的影响, 丢失或重新排序数据包都会导致所有活动的事务都陷入停滞。

### 1.2 Delegation to QUIC

The QUIC transport protocol incorporates stream multiplexing and per-stream flow control, similar to that provided by the HTTP/2 framing layer. By providing reliability at the stream level and congestion control across the entire connection, QUIC has the capability to improve the performance of HTTP compared to a TCP mapping. QUIC also incorporates TLS 1.3 ([[TLS](https://www.rfc-editor.org/rfc/rfc9114.html#TLS)]) at the transport layer, offering comparable confidentiality and integrity to running TLS over TCP, with the improved connection setup latency of TCP Fast Open ([[TFO](https://www.rfc-editor.org/rfc/rfc9114.html#TFO)]).

QUIC传输协议结合了流的多路复用和流量控制机制, 类似于HTTP/2的成帧层。通过在整个连接上提供流级别的可靠性和拥塞控制，QUIC可以提升基于TCP的HTTP性能。QUIC还在传输层集成了TLS 1.3([[TLS](https://www.rfc-editor.org/rfc/rfc9114.html#TLS)])，相较于在TCP上运行的TLS实现了机密性和完整性，并改善了TCP Fast Open([[TFO](https://www.rfc-editor.org/rfc/rfc9114.html#TFO)])的连接建立延迟。

This document defines HTTP/3: a mapping of HTTP semantics over the QUIC transport protocol, drawing heavily on the design of HTTP/2. HTTP/3 relies on QUIC to provide confidentiality and integrity protection of data; peer authentication; and reliable, in-order, per-stream delivery. While delegating stream lifetime and flow-control issues to QUIC, a binary framing similar to the HTTP/2 framing is used on each stream. Some HTTP/2 features are subsumed by QUIC, while other features are implemented atop QUIC.

本文档定义的HTTP/3: HTTP语义在QUIC传输协议上的映射，在很大程度上依赖于HTTP/2的设计。HTTP/3依赖QUIC提供的数据机密性和完整性保护；对等认证；以及可靠, 有序的流传递。当流的生命周期和流量控制问题依托于QUIC时, 每个流会使用类似于HTTP/2的二进制帧。QUIC包含HTTP/2的一些特性, 其他特性则在QUIC上实现。

QUIC is described in [[QUIC-TRANSPORT](https://www.rfc-editor.org/rfc/rfc9114.html#QUIC-TRANSPORT)]. For a full description of HTTP/2, see [[HTTP/2](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9113)].

有关QUIC的描述见: [[QUIC-TRANSPORT](https://www.rfc-editor.org/rfc/rfc9114.html#QUIC-TRANSPORT)]。有关HTTP/2的完整描述见:  [[HTTP/2](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9113)].

## 2. HTTP/3 协议概览

HTTP/3 provides a transport for HTTP semantics using the QUIC transport protocol and an internal framing layer similar to HTTP/2.

HTTP/3提供的HTTP语义使用了和HTTP/2二进制帧层面类似的QUIC传输协议。

Once a client knows that an HTTP/3 server exists at a certain endpoint, it opens a QUIC connection. QUIC provides protocol negotiation, stream-based multiplexing, and flow control. Discovery of an HTTP/3 endpoint is described in [Section 3.1](https://www.rfc-editor.org/rfc/rfc9114.html#discovery).

一旦客户端知道在某个端点存在HTTP/3服务, 就会打开一个QUIC连接。QUIC提供协商协议, 基于流的多路复用和流量控制。在 [Section 3.1](https://www.rfc-editor.org/rfc/rfc9114.html#discovery)中描述了如何发现一个HTTP/3的端点。

Within each stream, the basic unit of HTTP/3 communication is a frame ([Section 7.2](https://www.rfc-editor.org/rfc/rfc9114.html#frames)). Each frame type serves a different purpose. For example, [HEADERS](https://www.rfc-editor.org/rfc/rfc9114.html#frame-headers) and [DATA](https://www.rfc-editor.org/rfc/rfc9114.html#frame-data) frames form the basis of HTTP requests and responses ([Section 4.1](https://www.rfc-editor.org/rfc/rfc9114.html#request-response)). Frames that apply to the entire connection are conveyed on a dedicated [control stream](https://www.rfc-editor.org/rfc/rfc9114.html#control-streams).

在每个流中，HTTP/3通信的基本单元是帧([Section 7.2](https://www.rfc-editor.org/rfc/rfc9114.html#frames))。每种帧类型都有不同的用途。比如 [HEADERS](https://www.rfc-editor.org/rfc/rfc9114.html#frame-headers) 和 [DATA](https://www.rfc-editor.org/rfc/rfc9114.html#frame-data) 帧构成了请求和响应的基础([Section 4.1](https://www.rfc-editor.org/rfc/rfc9114.html#request-response))。应用于整个连接的帧在专用的[控制流](https://www.rfc-editor.org/rfc/rfc9114.html#control-streams)中传输。

Multiplexing of requests is performed using the QUIC stream abstraction, which is described in [Section 2](https://www.rfc-editor.org/rfc/rfc9000#section-2) of [[QUIC-TRANSPORT](https://www.rfc-editor.org/rfc/rfc9114.html#QUIC-TRANSPORT)]. Each request-response pair consumes a single QUIC stream. Streams are independent of each other, so one stream that is blocked or suffers packet loss does not prevent progress on other streams.

请求的多路传输是使用QUIC流来抽象执行的, 在 [QUIC](https://www.rfc-editor.org/rfc/rfc9114.html#QUIC-TRANSPORT) 传输协议的[第二节](https://www.rfc-editor.org/rfc/rfc9000#section-2)中有相关描述。每个请求-响应使用单独数据流, 流相互独立, 因此一个流的阻塞或丢包并不影响其他流的处理。

Server push is an interaction mode introduced in HTTP/2 ([[HTTP/2](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9113)]) that permits a server to push a request-response exchange to a client in anticipation of the client making the indicated request. This trades off network usage against a potential latency gain. Several HTTP/3 frames are used to manage server push, such as [PUSH_PROMISE](https://www.rfc-editor.org/rfc/rfc9114.html#frame-push-promise), [MAX_PUSH_ID](https://www.rfc-editor.org/rfc/rfc9114.html#frame-max-push-id), and [CANCEL_PUSH](https://www.rfc-editor.org/rfc/rfc9114.html#frame-cancel-push).

服务器推送是在[HTTP/2](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9113)中引入的一种交互模式, 它允许服务器在预期客户端发出指定请求的情况下将响应数据推送到客户端。这样在网络使用率与潜在延迟增加之前做了权衡。有几个HTTP/3的帧用来管理服务推送, 如:  [PUSH_PROMISE](https://www.rfc-editor.org/rfc/rfc9114.html#frame-push-promise),  [MAX_PUSH_ID](https://www.rfc-editor.org/rfc/rfc9114.html#frame-max-push-id), [CANCEL_PUSH](https://www.rfc-editor.org/rfc/rfc9114.html#frame-cancel-push)。

As in HTTP/2, request and response fields are compressed for transmission. Because HPACK ([[HPACK](https://www.rfc-editor.org/rfc/rfc9114.html#HPACK)]) relies on in-order transmission of compressed field sections (a guarantee not provided by QUIC), HTTP/3 replaces HPACK with QPACK ([[QPACK](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9204)]). QPACK uses separate unidirectional streams to modify and track field table state, while encoded field sections refer to the state of the table without modifying it.

与HTTP/2一样, 请求和响应字段在传输时将被压缩。由于[HPACK](https://www.rfc-editor.org/rfc/rfc9114.html#HPACK)依赖压缩字段的有序传输(QUIC没有提供相关的保证), 因此HTTP/3使用[QPACK](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9204)取代了HPACK。QPACK使用单独的单向流来修改和跟踪字段表状态，编码部分的字段引用表的状态而非修改它。

### 2.1 文档组织

The following sections provide a detailed overview of the lifecycle of an HTTP/3 connection:

- "[Connection Setup and Management](https://www.rfc-editor.org/rfc/rfc9114.html#connection-setup)" ([Section 3](https://www.rfc-editor.org/rfc/rfc9114.html#connection-setup)) covers how an HTTP/3 endpoint is discovered and an HTTP/3 connection is established.
- "[Expressing HTTP Semantics in HTTP/3](https://www.rfc-editor.org/rfc/rfc9114.html#http-request-lifecycle)" ([Section 4](https://www.rfc-editor.org/rfc/rfc9114.html#http-request-lifecycle)) describes how HTTP semantics are expressed using frames.
- "[Connection Closure](https://www.rfc-editor.org/rfc/rfc9114.html#connection-closure)" ([Section 5](https://www.rfc-editor.org/rfc/rfc9114.html#connection-closure)) describes how HTTP/3 connections are terminated, either gracefully or abruptly.

以下各部分详细阐述了HTTP/3连接的生命周期:

- "[连接设置和管理](https://www.rfc-editor.org/rfc/rfc9114.html#connection-setup)" ([Section 3](https://www.rfc-editor.org/rfc/rfc9114.html#connection-setup)) 介绍了如何发现HTTP/3节点以及如何建立HTTP/3连接。
- "[HTTP/3中的HTTP语义表达](https://www.rfc-editor.org/rfc/rfc9114.html#http-request-lifecycle)"  ([Section 4](https://www.rfc-editor.org/rfc/rfc9114.html#http-request-lifecycle)) 介绍了如何使用帧表达HTTP语义。
- "[连接关闭](https://www.rfc-editor.org/rfc/rfc9114.html#connection-closure)" ([Section 5](https://www.rfc-editor.org/rfc/rfc9114.html#connection-closure)) 介绍如何优雅, 或者遇到异常冲突时关闭连接。

The details of the wire protocol and interactions with the transport are described in subsequent sections:

- "[Stream Mapping and Usage](https://www.rfc-editor.org/rfc/rfc9114.html#stream-mapping)" ([Section 6](https://www.rfc-editor.org/rfc/rfc9114.html#stream-mapping)) describes the way QUIC streams are used.
- "[HTTP Framing Layer](https://www.rfc-editor.org/rfc/rfc9114.html#http-framing-layer)" ([Section 7](https://www.rfc-editor.org/rfc/rfc9114.html#http-framing-layer)) describes the frames used on most streams.
- "[Error Handling](https://www.rfc-editor.org/rfc/rfc9114.html#errors)" ([Section 8](https://www.rfc-editor.org/rfc/rfc9114.html#errors)) describes how error conditions are handled and expressed, either on a particular stream or for the connection as a whole.

协议的详细信息以及传输交互方式将在以下章节做介绍:

- "[流的映射和使用](https://www.rfc-editor.org/rfc/rfc9114.html#stream-mapping)" ([Section 6](https://www.rfc-editor.org/rfc/rfc9114.html#stream-mapping)) 介绍QUIC流的使用方式。
- "[HTTP成帧层](https://www.rfc-editor.org/rfc/rfc9114.html#http-framing-layer)" ([Section 7](https://www.rfc-editor.org/rfc/rfc9114.html#http-framing-layer)) 描述了在大多数流中使用的帧。
- "[错误处理](https://www.rfc-editor.org/rfc/rfc9114.html#errors)" ([Section 8](https://www.rfc-editor.org/rfc/rfc9114.html#errors)) 描述了如何在特定或整个连接上表示和处理错误。

Additional resources are provided in the final sections:

- "[Extensions to HTTP/3](https://www.rfc-editor.org/rfc/rfc9114.html#extensions)" ([Section 9](https://www.rfc-editor.org/rfc/rfc9114.html#extensions)) describes how new capabilities can be added in future documents.
- A more detailed comparison between HTTP/2 and HTTP/3 can be found in [Appendix A](https://www.rfc-editor.org/rfc/rfc9114.html#h2-considerations).

最后几节提供了额外的资源:

- "[HTTP/3扩展](https://www.rfc-editor.org/rfc/rfc9114.html#extensions)" ([Section 9](https://www.rfc-editor.org/rfc/rfc9114.html#extensions)) 描述了如何在未来的文档中添加新功能。
- 关于HTTP/2和HTTP/3更详细的对比信息可以参见[附录A](https://www.rfc-editor.org/rfc/rfc9114.html#h2-considerations)

### 2.2 约定和术语

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**", "**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document are to be interpreted as described in BCP 14 [[RFC2119](https://www.rfc-editor.org/rfc/rfc9114.html#RFC2119)] [[RFC8174](https://www.rfc-editor.org/rfc/rfc9114.html#RFC8174)] when, and only when, they appear in all capitals, as shown here.

关键词 "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**", "**NOT RECOMMENDED**", "**MAY**", "**OPTIONAL**" 如此处所示, 当且仅当这些词以所有的大写字母出现时, 依据BCP 14 [[RFC2119](https://www.rfc-editor.org/rfc/rfc9114.html#RFC2119)] [[RFC8174](https://www.rfc-editor.org/rfc/rfc9114.html#RFC8174)] 解释。

This document uses the variable-length integer encoding from [[QUIC-TRANSPORT](https://www.rfc-editor.org/rfc/rfc9114.html#QUIC-TRANSPORT)].

本文档使用 [[QUIC-TRANSPORT](https://www.rfc-editor.org/rfc/rfc9114.html#QUIC-TRANSPORT)]中可变长度整数编码。

The following terms are used:

- abort: An abrupt termination of a connection or stream, possibly due to an error condition

- client: The endpoint that initiates an HTTP/3 connection. Clients send HTTP requests and receive HTTP responses.
- connection: A transport-layer connection between two endpoints using QUIC as the transport protocol.
- [connection error](https://www.rfc-editor.org/rfc/rfc9114.html#errors): An error that affects the entire HTTP/3 connection.
- endpoint: Either the client or server of the connection.
- frame: The smallest unit of communication on a stream in HTTP/3, consisting of a header and a variable-length sequence of bytes structured according to the frame type.Protocol elements called "frames" exist in both this document and [[QUIC-TRANSPORT](https://www.rfc-editor.org/rfc/rfc9114.html#QUIC-TRANSPORT)]. Where frames from [[QUIC-TRANSPORT](https://www.rfc-editor.org/rfc/rfc9114.html#QUIC-TRANSPORT)] are referenced, the frame name will be prefaced with "QUIC". For example, "QUIC CONNECTION_CLOSE frames". References without this preface refer to frames defined in [Section 7.2](https://www.rfc-editor.org/rfc/rfc9114.html#frames).
- HTTP/3 connection: A QUIC connection where the negotiated application protocol is HTTP/3.
- peer: An endpoint. When discussing a particular endpoint, "peer" refers to the endpoint that is remote to the primary subject of discussion.
- receiver: An endpoint that is receiving frames.
- sender: An endpoint that is transmitting frames.
- server: The endpoint that accepts an HTTP/3 connection. Servers receive HTTP requests and send HTTP responses.
- stream: A bidirectional or unidirectional bytestream provided by the QUIC transport. All streams within an HTTP/3 connection can be considered "HTTP/3 streams", but multiple stream types are defined within HTTP/3.
- [stream error](https://www.rfc-editor.org/rfc/rfc9114.html#errors): An application-level error on the individual stream.

以下术语被使用:

- abort: 可能由于错误的条件, 连接或流突然终止。
- client: 启动HTTP/3连接的端点。客户发送HTTP请求并接收HTTP响应。
- connection: 使用QUIC作为两个端点的传输层协议。
- connection error：影响整个HTTP/3连接的错误。
- endpoint：通信端点, 连接的客户端或服务器。
- frame: 数据帧, HTTP/3中流上的最小通信单元，由HEADER和根据帧类型构造的可变长度序列组成。本文档和[Quic-Transport](https://www.rfc-editor.org/rfc/rfc9114.html#QUIC-TRANSPORT)中都存在称为“帧”的协议元素。在引用来自[Quic-Transport](https://www.rfc-editor.org/rfc/rfc9114.html#QUIC-TRANSPORT)的帧时，帧名称将以“QUIC”开头。例如，"QUIC CONNECTION_CLOSE Frame”。没有QUIC开头的帧名称参考文档[第7.2节](https://www.rfc-editor.org/rfc/rfc9114.html#frames)中的定义。
- HTTP/3 connection: 协程应用协议为HTTP/3的QUIC连接。
- peer: 一个通信端点, 在讨论一个特别的通信端点时, "peer"为连接此端点的远程端点。
- receiver: 正在接收帧的通信端点。
- sender: 正在发送帧的通信端点。
- server: 接受HTTP/3连接的端点。服务器接收HTTP请求并发送HTTP响应。
- stream: QUIC传输提供的双向或单向字节流。一个HTTP/3连接中的所有流都可以被视为“HTTP/3流”，但在HTTP/3中定义了多种流类型。
- [stream error](https://www.rfc-editor.org/rfc/rfc9114.html#errors): 单个流上的应用层错误。

The term "content" is defined in [Section 6.4](https://www.rfc-editor.org/rfc/rfc9110#section-6.4) of [[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)].

Finally, the terms "resource", "message", "user agent", "origin server", "gateway", "intermediary", "proxy", and "tunnel" are defined in [Section 3](https://www.rfc-editor.org/rfc/rfc9110#section-3) of [[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)].

Packet diagrams in this document use the format defined in [Section 1.3](https://www.rfc-editor.org/rfc/rfc9000#section-1.3) of [[QUIC-TRANSPORT](https://www.rfc-editor.org/rfc/rfc9114.html#QUIC-TRANSPORT)] to illustrate the order and size of fields.

术语内容在 [[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)]的[6.4节中定义](https://www.rfc-editor.org/rfc/rfc9110#section-6.4)。

最后, 在 [[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)] [第3节](https://www.rfc-editor.org/rfc/rfc9110#section-3) 定义了术语 "resource", "message", "user agent", "origin server", "gateway", "intermediary", "proxy",  "tunnel"。

本文件中的数据包图使用[QUIC-TRANSPORT]((https://www.rfc-editor.org/rfc/rfc9114.html#QUIC-TRANSPORT)) [第1.3节](https://www.rfc-editor.org/rfc/rfc9000#section-1.3)中定义的格式来说明字段的顺序和大小。

## 3. 连接的设置和管理

### 3.1 发现HTTP/3端点

HTTP relies on the notion of an authoritative response: a response that has been determined to be the most appropriate response for that request given the state of the target resource at the time of response message origination by (or at the direction of) the origin server identified within the target URI. Locating an authoritative server for an HTTP URI is discussed in [Section 4.3](https://www.rfc-editor.org/rfc/rfc9110#section-4.3) of [[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)].

HTTP依赖于权威响应的概念：在目标URI内标识的源服务器发起响应消息时(或在其指示下)，给定目标资源在响应消息发起时的状态，已经确定对该请求最合适的响应。在[HTTP]的第4.3节中讨论了为HTTP URI定位权威服务器。

The "https" scheme associates authority with possession of a certificate that the client considers to be trustworthy for the host identified by the authority component of the URI. Upon receiving a server certificate in the TLS handshake, the client **MUST** verify that the certificate is an acceptable match for the URI's origin server using the process described in [Section 4.3.4](https://www.rfc-editor.org/rfc/rfc9110#section-4.3.4) of [[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)]. If the certificate cannot be verified with respect to the URI's origin server, the client **MUST NOT** consider the server authoritative for that origin.

“HTTPS”方案将授权与拥有证书相关联，客户端认为该证书对于由URI的授权组件标识的主机是可信的。在TLS握手中收到服务器证书后，客户端必须使用[HTTP]第4.3.4节中描述的过程来验证该证书对于URI的源服务器是可接受的匹配。如果证书不能相对于URI的源服务器进行验证，则客户端不能认为该服务器对该源具有权威性。

A client **MAY** attempt access to a resource with an "https" URI by resolving the host identifier to an IP address, establishing a QUIC connection to that address on the indicated port (including validation of the server certificate as described above), and sending an HTTP/3 request message targeting the URI to the server over that secured connection. Unless some other mechanism is used to select HTTP/3, the token "h3" is used in the Application-Layer Protocol Negotiation (ALPN; see [[RFC7301](https://www.rfc-editor.org/rfc/rfc9114.html#RFC7301)]) extension during the TLS handshake.

客户端可以通过将主机标识符解析为IP地址，在所指示的端口上建立到该地址的Quic连接(包括如上所述的服务器证书的验证)，并通过该安全连接向服务器发送针对该URI的HTTP/3请求消息，来尝试访问具有“HTTPS”URI的资源。除非使用某种其他机制来选择HTTP/3，否则在TLS握手期间，令牌“h3”用于应用层协议协商(ALPN；见[RFC7301])扩展。

Connectivity problems (e.g., blocking UDP) can result in a failure to establish a QUIC connection; clients **SHOULD** attempt to use TCP-based versions of HTTP in this case.

连接问题（例如，阻止UDP）可能导致无法建立Quic连接；在这种情况下，客户应尝试使用基于TCP的HTTP版本。

Servers **MAY** serve HTTP/3 on any UDP port; an alternative service advertisement always includes an explicit port, and URIs contain either an explicit port or a default port associated with the scheme.

服务器可以在任何 UDP 端口上提供 HTTP/3；一个可选服务始终包含显式端口，并且 URI 包含与方案关联的显式端口或默认端口。

#### 3.1.1 HTTP 可选服务

An HTTP origin can advertise the availability of an equivalent HTTP/3 endpoint via the Alt-Svc HTTP response header field or the HTTP/2 ALTSVC frame ([[ALTSVC](https://www.rfc-editor.org/rfc/rfc9114.html#ALTSVC)]) using the "h3" ALPN token.

For example, an origin could indicate in an HTTP response that HTTP/3 was available on UDP port 50781 at the same hostname by including the following header field:

HTTP源可以在HTTP响应头`Alt-Svc`字段或者HTTP/2`ALTSVC`帧([[ALTSVC](https://www.rfc-editor.org/rfc/rfc9114.html#ALTSVC)]) 中, 使用`h3` ALPN令牌通告HTTP/3端点的可用性。

例如, 服务源可以在 HTTP 响应中表示 HTTP/3 在同一主机的 UDP 端口 50781 上可用, 方法是包含以下响应头字段: 

```http-message
Alt-Svc: h3=":50781"
```

On receipt of an Alt-Svc record indicating HTTP/3 support, a client **MAY** attempt to establish a QUIC connection to the indicated host and port; if this connection is successful, the client can send HTTP requests using the mapping described in this document.

在收到指示 HTTP/3 支持的 Alt-Svc 记录后，客户端可能会尝试建立与指示的主机和端口的 QUIC 连接； 如果此连接成功，客户端可以使用本文档中描述的映射发送 HTTP 请求。

#### 3.1.2 其他方案

Although HTTP is independent of the transport protocol, the "http" scheme associates authority with the ability to receive TCP connections on the indicated port of whatever host is identified within the authority component. Because HTTP/3 does not use TCP, HTTP/3 cannot be used for direct access to the authoritative server for a resource identified by an "http" URI. However, protocol extensions such as [[ALTSVC](https://www.rfc-editor.org/rfc/rfc9114.html#ALTSVC)] permit the authoritative server to identify other services that are also authoritative and that might be reachable over HTTP/3.

尽管HTTP独立于传输协议，但“http”方案将授权与在授权组件中标识的任何主机的指示端口上接收TCP连接的能力相关联。因为 HTTP/3 不使用 TCP，所以 HTTP/3 不能用于直接访问由“http”URI 资源标识的权威服务器。 但是，诸如 [[ALTSVC](https://www.rfc-editor.org/rfc/rfc9114.html#ALTSVC)] 之类的协议扩展允许权威服务器识别其他同样具有权威性并且可以通过 HTTP/3 访问的服务。

Prior to making requests for an origin whose scheme is not "https", the client **MUST** ensure the server is willing to serve that scheme. For origins whose scheme is "http", an experimental method to accomplish this is described in [[RFC8164](https://www.rfc-editor.org/rfc/rfc9114.html#RFC8164)]. Other mechanisms might be defined for various schemes in the future.

在向方案不是“https”的源发出请求之前，客户端必须确保服务器能够为该方案提供服务。 对于方案为“http”的源，在 [[RFC8164](https://www.rfc-editor.org/rfc/rfc9114.html#RFC8164)] 中描述了实现此目的的实验方法。 将来可能会为各种方案定义其他机制。

### 3.2 连接建立

HTTP/3 relies on QUIC version 1 as the underlying transport. The use of other QUIC transport versions with HTTP/3 **MAY** be defined by future specifications.

QUIC version 1 uses TLS version 1.3 or greater as its handshake protocol. HTTP/3 clients **MUST** support a mechanism to indicate the target host to the server during the TLS handshake. If the server is identified by a domain name ([[DNS-TERMS](https://www.rfc-editor.org/rfc/rfc9114.html#DNS-TERMS)]), clients **MUST** send the Server Name Indication (SNI; [[RFC6066](https://www.rfc-editor.org/rfc/rfc9114.html#RFC6066)]) TLS extension unless an alternative mechanism to indicate the target host is used.

QUIC connections are established as described in [[QUIC-TRANSPORT](https://www.rfc-editor.org/rfc/rfc9114.html#QUIC-TRANSPORT)]. During connection establishment, HTTP/3 support is indicated by selecting the ALPN token "h3" in the TLS handshake. Support for other application-layer protocols **MAY** be offered in the same handshake.

While connection-level options pertaining to the core QUIC protocol are set in the initial crypto handshake, settings specific to HTTP/3 are conveyed in the [SETTINGS](https://www.rfc-editor.org/rfc/rfc9114.html#frame-settings) frame. After the QUIC connection is established, a [SETTINGS](https://www.rfc-editor.org/rfc/rfc9114.html#frame-settings) frame **MUST** be sent by each endpoint as the initial frame of their respective HTTP [control stream](https://www.rfc-editor.org/rfc/rfc9114.html#control-streams).

HTTP/3 依赖于 QUIC 版本 1 作为底层传输。 未来规划 HTTP/3 可能会使用其他 QUIC 传输版本。 HTTP/3 客户端必须支持在 TLS 握手期间向服务器指示目标主机的机制。如果服务器由域名 ([[DNS-TERMS](https://www.rfc-editor.org/rfc/rfc9114.html#DNS-TERMS)]) 标识，则客户端必须发送服务器名称指示 (SNI;  [[RFC6066](https://www.rfc-editor.org/rfc/rfc9114.html#RFC6066)]) TLS 扩展，除非使用了指示目标主机的替代机制。

QUIC 连接按照 [[QUIC-TRANSPORT](https://www.rfc-editor.org/rfc/rfc9114.html#QUIC-TRANSPORT)] 中的描述建立。在连接建立期间，通过在 TLS 握手中选择 ALPN 令牌“h3”来指示 HTTP/3 支持。可以在同一握手中提供对其他应用层协议的支持。

虽然与核心 QUIC 协议有关的连接级别选项是在初始加密握手中设置的，但 HTTP/3 特定设置在 SETTINGS 帧中传达。 QUIC 连接建立后，每个端点必须发送一个  [SETTINGS](https://www.rfc-editor.org/rfc/rfc9114.html#frame-settings)  帧（第 7.2.4 节）作为各自 HTTP 控制流的初始帧。

### 3.3 连接重用

HTTP/3 connections are persistent across multiple requests. For best performance, it is expected that clients will not close connections until it is determined that no further communication with a server is necessary (for example, when a user navigates away from a particular web page) or until the server closes the connection.

HTTP/3连接在多个请求之间是持久的。为了获得最佳性能，客户端将不会关闭连接，直到确定不需要与服务器进行进一步通信（例如，当用户导航离开特定网页时）或直到服务器关闭连接。

Once a connection to a server endpoint exists, this connection **MAY** be reused for requests with multiple different URI authority components. To use an existing connection for a new origin, clients **MUST** validate the certificate presented by the server for the new origin server using the process described in [Section 4.3.4](https://www.rfc-editor.org/rfc/rfc9110#section-4.3.4) of [[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)]. This implies that clients will need to retain the server certificate and any additional information needed to verify that certificate; clients that do not do so will be unable to reuse the connection for additional origins.

一旦存在与服务器端点的连接，则可以重复使用此连接，以用于多个不同URI的请求。要使用新来源的现有连接，客户端必须使用 [[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)]第[Section 4.3.4](https://www.rfc-editor.org/rfc/rfc9110#section-4.3.4)节中描述的过程验证服务器为新Origin服务器提供的证书。这意味着客户端将需要保留服务器证书和验证该证书所需的任何其他信息；不这样做的客户端将无法在其他新的源中重复使用此连接。

If the certificate is not acceptable with regard to the new origin for any reason, the connection **MUST NOT** be reused and a new connection **SHOULD** be established for the new origin. If the reason the certificate cannot be verified might apply to other origins already associated with the connection, the client **SHOULD** revalidate the server certificate for those origins. For instance, if validation of a certificate fails because the certificate has expired or been revoked, this might be used to invalidate all other origins for which that certificate was used to establish authority.

如果对新来源的证书不可接受, 不管出于任何原因, 都不得重复使用该连接, 并且应为新来源建立新的连接。如果无法验证证书的情形可能也存在与该连接关联的其他源, 则客户端应该重新验证这些源的服务器证书。例如，如果证书的验证因证书已过期或已撤销而失败，则使用该证书建立权限认证的所有其他起源无效。

Clients **SHOULD NOT** open more than one HTTP/3 connection to a given IP address and UDP port, where the IP address and port might be derived from a URI, a selected alternative service ([[ALTSVC](https://www.rfc-editor.org/rfc/rfc9114.html#ALTSVC)]), a configured proxy, or name resolution of any of these. A client **MAY** open multiple HTTP/3 connections to the same IP address and UDP port using different transport or TLS configurations but **SHOULD** avoid creating multiple connections with the same configuration.

客户端不应打开一个以上的http/3连接到给定的IP地址和UDP端口，其中IP地址和端口可能是派生于URI，已被选择的替代服务（[[ALTSVC](https://www.rfc-editor.org/rfc/rfc9114.html#ALTSVC)]），已配置的代理或其中任何一个的名称解析。客户端可以使用不同的传输或TLS配置打开多个HTTP/3连接到相同的IP地址和UDP端口，但应避免使用相同的配置创建多个连接。

Servers are encouraged to maintain open HTTP/3 connections for as long as possible but are permitted to terminate idle connections if necessary. When either endpoint chooses to close the HTTP/3 connection, the terminating endpoint **SHOULD** first send a [GOAWAY](https://www.rfc-editor.org/rfc/rfc9114.html#frame-goaway) frame ([Section 5.2](https://www.rfc-editor.org/rfc/rfc9114.html#connection-shutdown)) so that both endpoints can reliably determine whether previously sent frames have been processed and gracefully complete or terminate any necessary remaining tasks.

鼓励服务器尽可能长时间保持开放的HTTP/3连接，但在必要时允许终止空闲连接。当任一端点选择关闭HTTP/3连接时，终端端点应首先发送  [GOAWAY](https://www.rfc-editor.org/rfc/rfc9114.html#frame-goaway) 帧 ([Section 5.2](https://www.rfc-editor.org/rfc/rfc9114.html#connection-shutdown))，以便两个端点都可以确定是否已经可靠的处理过并优雅地完剩余任务。 

A server that does not wish clients to reuse HTTP/3 connections for a particular origin can indicate that it is not authoritative for a request by sending a 421 (Misdirected Request) status code in response to the request; see [Section 7.4](https://www.rfc-editor.org/rfc/rfc9110#section-7.4) of [[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)].

不希望客户端重新使用特定来源的HTTP/3连接，特定来源指: 可以通过发送421(错误定向的请求)状态代码来响应请求，以表明它对请求没有权限认证；请参阅[[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)]的第[7.4](https://www.rfc-editor.org/rfc/rfc9110#section-7.4)节。

## 4. HTTP/3中的HTTP语义表示

#### 4.1. HTTP消息帧

A client sends an HTTP request on a [request stream](https://www.rfc-editor.org/rfc/rfc9114.html#request-streams), which is a client-initiated bidirectional QUIC stream; see [Section 6.1](https://www.rfc-editor.org/rfc/rfc9114.html#request-streams). A client **MUST** send only a single request on a given stream. A server sends zero or more interim HTTP responses on the same stream as the request, followed by a single final HTTP response, as detailed below. See [Section 15](https://www.rfc-editor.org/rfc/rfc9110#section-15) of [[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)] for a description of interim and final HTTP responses.

Pushed responses are sent on a server-initiated unidirectional QUIC stream; see [Section 6.2.2](https://www.rfc-editor.org/rfc/rfc9114.html#push-streams). A server sends zero or more interim HTTP responses, followed by a single final HTTP response, in the same manner as a standard response. Push is described in more detail in [Section 4.6](https://www.rfc-editor.org/rfc/rfc9114.html#server-push).

On a given stream, receipt of multiple requests or receipt of an additional HTTP response following a final HTTP response **MUST** be treated as [malformed](https://www.rfc-editor.org/rfc/rfc9114.html#malformed).

An HTTP message (request or response) consists of:

1. the header section, including message control data, sent as a single [HEADERS](https://www.rfc-editor.org/rfc/rfc9114.html#frame-headers) frame,
2. optionally, the content, if present, sent as a series of [DATA](https://www.rfc-editor.org/rfc/rfc9114.html#frame-data) frames, and
3. optionally, the trailer section, if present, sent as a single [HEADERS](https://www.rfc-editor.org/rfc/rfc9114.html#frame-headers) frame.

客户端在请求流上发送HTTP请求，该请求流是客户端发射的双向Quic流；请参阅第[Section 6.1](https://www.rfc-editor.org/rfc/rfc9114.html#request-streams)节。客户端必须仅在给定的流上发送一个请求。服务器在请求的同一流中发送零或更多的临时HTTP响应，随后是单个最终HTTP响应，详情参见 [[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)]的第 [15](https://www.rfc-editor.org/rfc/rfc9110#section-15)节，描述了有关中期和最终响应的描述。

推送响应在服务器启动的在单向Quic流上被发送；请参阅第[6.2.2](https://www.rfc-editor.org/rfc/rfc9114.html#push-streams)节。服务器以相同的方式发送零或更多的临时HTTP标准响应，然后是单个最终HTTP响应。在第[Section 4.6](https://www.rfc-editor.org/rfc/rfc9114.html#server-push)节中更详细地描述了推动。

给定的流中，在最终的HTTP响应之后接受多个或者额外的请求, 必须被视为格式错误。

HTTP消息（请求或响应）包括：

1.  header部分, 作为一个 [HEADERS](https://www.rfc-editor.org/rfc/rfc9114.html#frame-headers) 帧被发送,  包括消息控制数据的报头部分

2. 可选的内容, 如果存在, 作为一系列 [DATA](https://www.rfc-editor.org/rfc/rfc9114.html#frame-data) 帧发送

3. 可选的预告部分, 如果存在, 作为单独的 [HEADERS](https://www.rfc-editor.org/rfc/rfc9114.html#frame-headers) 帧发送

Header and trailer sections are described in Sections [6.3](https://www.rfc-editor.org/rfc/rfc9110#section-6.3) and [6.5](https://www.rfc-editor.org/rfc/rfc9110#section-6.5) of [[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)]; the content is described in [Section 6.4](https://www.rfc-editor.org/rfc/rfc9110#section-6.4) of [[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)].

Receipt of an invalid sequence of frames **MUST** be treated as a [connection error](https://www.rfc-editor.org/rfc/rfc9114.html#errors) of type [H3_FRAME_UNEXPECTED](https://www.rfc-editor.org/rfc/rfc9114.html#H3_FRAME_UNEXPECTED). In particular, a [DATA](https://www.rfc-editor.org/rfc/rfc9114.html#frame-data) frame before any [HEADERS](https://www.rfc-editor.org/rfc/rfc9114.html#frame-headers) frame, or a [HEADERS](https://www.rfc-editor.org/rfc/rfc9114.html#frame-headers) or [DATA](https://www.rfc-editor.org/rfc/rfc9114.html#frame-data) frame after the trailing [HEADERS](https://www.rfc-editor.org/rfc/rfc9114.html#frame-headers) frame, is considered invalid. Other frame types, especially unknown frame types, might be permitted subject to their own rules; see [Section 9](https://www.rfc-editor.org/rfc/rfc9114.html#extensions).

A server **MAY** send one or more [PUSH_PROMISE](https://www.rfc-editor.org/rfc/rfc9114.html#frame-push-promise) frames before, after, or interleaved with the frames of a response message. These [PUSH_PROMISE](https://www.rfc-editor.org/rfc/rfc9114.html#frame-push-promise) frames are not part of the response; see [Section 4.6](https://www.rfc-editor.org/rfc/rfc9114.html#server-push) for more details. [PUSH_PROMISE](https://www.rfc-editor.org/rfc/rfc9114.html#frame-push-promise) frames are not permitted on [push streams](https://www.rfc-editor.org/rfc/rfc9114.html#push-streams); a pushed response that includes [PUSH_PROMISE](https://www.rfc-editor.org/rfc/rfc9114.html#frame-push-promise) frames **MUST** be treated as a [connection error](https://www.rfc-editor.org/rfc/rfc9114.html#errors) of type [H3_FRAME_UNEXPECTED](https://www.rfc-editor.org/rfc/rfc9114.html#H3_FRAME_UNEXPECTED).

Frames of unknown types ([Section 9](https://www.rfc-editor.org/rfc/rfc9114.html#extensions)), including reserved frames ([Section 7.2.8](https://www.rfc-editor.org/rfc/rfc9114.html#frame-reserved)) **MAY** be sent on a request or [push stream](https://www.rfc-editor.org/rfc/rfc9114.html#push-streams) before, after, or interleaved with other frames described in this section.

Header头和预告在[[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)]的 [6.3](https://www.rfc-editor.org/rfc/rfc9110#section-6.3)和 [6.5](https://www.rfc-editor.org/rfc/rfc9110#section-6.5)节中进行了描述；内容部分在[[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)]的第[Section 6.4](https://www.rfc-editor.org/rfc/rfc9110#section-6.4) 节中进行了描述。

接受无效序列的帧必须被视为 H3_FRAME_UNEXPECT 类型的连接错误。特别是当 DATA 帧在任何 HEADER 帧之前, 或者 HEADER / DATA 帧出现在结尾的 HEADER 帧之后,被认为是无效的。其他帧类型，尤其是未知帧类型，可能会根据其自身的规则被认为是允许的；见[Section 9](https://www.rfc-editor.org/rfc/rfc9114.html#extensions)。

服务器可以在响应消息帧之前、之后或与响应消息帧交叉发送一个或多个PUSH_PROMISE帧。这些PUSH_PROMISE帧不是响应的一部分；有关更多详细信息，请参阅第 [Section 4.6](https://www.rfc-editor.org/rfc/rfc9114.html#server-push)节。推送流上不允许PUSH_PROMISE帧；包含PUSH_PROMISE帧的推送响应必须被视为H3_FRAME_UNEXPECTED类型的连接错误。

未知类型的帧（([Section 9](https://www.rfc-editor.org/rfc/rfc9114.html#extensions))），包括保留帧（[Section 7.2.8](https://www.rfc-editor.org/rfc/rfc9114.html#frame-reserved)），可以在请求或推送流之前，之后，或与本节中描述的其他帧交互发送。

The [HEADERS](https://www.rfc-editor.org/rfc/rfc9114.html#frame-headers) and [PUSH_PROMISE](https://www.rfc-editor.org/rfc/rfc9114.html#frame-push-promise) frames might reference updates to the QPACK dynamic table. While these updates are not directly part of the message exchange, they must be received and processed before the message can be consumed. See [Section 4.2](https://www.rfc-editor.org/rfc/rfc9114.html#header-formatting) for more details.

Transfer codings (see [Section 7](https://www.rfc-editor.org/rfc/rfc9112#section-7) of [[HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9112)]) are not defined for HTTP/3; the Transfer-Encoding header field **MUST NOT** be used.

A response **MAY** consist of multiple messages when and only when one or more interim responses (1xx; see [Section 15.2](https://www.rfc-editor.org/rfc/rfc9110#section-15.2) of [[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)]) precede a final response to the same request. Interim responses do not contain content or trailer sections.

HEADER 和 PUSH_PROMISE 帧可以引用对 QPACK 动态表的更新。虽然这些更新不是消息交换的直接部分，但必须在消息被使用之前接收和处理它们。详见 [Section 4.2](https://www.rfc-editor.org/rfc/rfc9114.html#header-formatting)。

传输编码（见[[HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9112)]第[Section 7](https://www.rfc-editor.org/rfc/rfc9112#section-7)节）在HTTP/3中未被定义；不得使用 Transfer-Encoding 头字段。

同一个请求中, 当且仅当一个或多个临时响应(1xx；参见 [[HTTP](https://www.rfc-editor.org/rfc/rfc9114.html#RFC9110)]第 [Section 15.2](https://www.rfc-editor.org/rfc/rfc9110#section-15.2) 节)先于最终响应时, 响应可能包含多个消息。临时响应不包含内容或预告部分。

An HTTP request/response exchange fully consumes a client-initiated bidirectional QUIC stream. After sending a request, a client **MUST** close the stream for sending. Unless using the CONNECT method (see [Section 4.4](https://www.rfc-editor.org/rfc/rfc9114.html#connect)), clients **MUST NOT** make stream closure dependent on receiving a response to their request. After sending a final response, the server **MUST** close the stream for sending. At this point, the QUIC stream is fully closed.

HTTP请求/响应交换完全承载于客户端发起的双向QUIC流。发送请求后，客户端必须关闭流以进行发送。除非使用 CONNECT 连接方法（参见第[Section 4.4](https://www.rfc-editor.org/rfc/rfc9114.html#connect)节），否则客户端不能依赖接受对于以其请求的响应使连接关闭。发送最终响应后，服务器必须关闭流才能继续发送。此时，QUIC流完全关闭。

When a stream is closed, this indicates the end of the final HTTP message. Because some messages are large or unbounded, endpoints **SHOULD** begin processing partial HTTP messages once enough of the message has been received to make progress. If a client-initiated stream terminates without enough of the HTTP message to provide a complete response, the server **SHOULD** abort its response stream with the error code [H3_REQUEST_INCOMPLETE](https://www.rfc-editor.org/rfc/rfc9114.html#H3_REQUEST_INCOMPLETE).

当流被关闭时，表示最后一条HTTP消息的结束。因为有些消息很大或是无限的，所以一旦接收到足够多的消息，端点就应该开始处理部分HTTP消息。如果客户端发起的流在没有足够的HTTP消息提供完整响应的情况下终止，则服务器应以错误代码H3_REQUEST_complete中止其响应流。

A server can send a complete response prior to the client sending an entire request if the response does not depend on any portion of the request that has not been sent and received. When the server does not need to receive the remainder of the request, it **MAY** abort reading the [request stream](https://www.rfc-editor.org/rfc/rfc9114.html#request-streams), send a complete response, and cleanly close the sending part of the stream. The error code [H3_NO_ERROR](https://www.rfc-editor.org/rfc/rfc9114.html#H3_NO_ERROR) **SHOULD** be used when requesting that the client stop sending on the [request stream](https://www.rfc-editor.org/rfc/rfc9114.html#request-streams). Clients **MUST NOT** discard complete responses as a result of having their request terminated abruptly, though clients can always discard responses at their discretion for other reasons. If the server sends a partial or complete response but does not abort reading the request, clients **SHOULD** continue sending the content of the request and close the stream normally.

如果响应不依赖于尚未发送和接收的请求的任何部分，则服务器可以在客户端发送整个请求之前发送完整响应。当服务器不需要接收请求的剩余部分时，它可以中止读取请求流，发送完整响应，并完全关闭流的发送部分。当客户端在 [请求流](https://www.rfc-editor.org/rfc/rfc9114.html#request-streams) 上停止发送时, 应该使用 [H3_NO_ERROR](https://www.rfc-editor.org/rfc/rfc9114.html#H3_NO_ERROR) 错误码。虽然客户端可以出于其他原因自行决定放弃响应, 但是客户端不得因其请求突然终止而放弃完整响应。如果服务器发送部分或完整响应，但没有中止读取请求，则客户端应继续发送请求内容并正常关闭流。

















