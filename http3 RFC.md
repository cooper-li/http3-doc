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

服务器可以在任何UDP端口上提供HTTP/3；备选服务公告始终包含显式端口，URI包含与方案关联的显式端口或默认端口。









