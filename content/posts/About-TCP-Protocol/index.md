---
title: About TCP Protocol
date: 2020-03-22
categories:
  - protocol

tags:
    - networking
    - tcp
---

## What is TCP ?

The Transmission Control Protocol (TCP) is one of the main protocols of the Internet protocol suite. It originated in the initial network implementation in which it complemented the Internet Protocol (IP). Therefore, the entire suite is commonly referred to as TCP/IP. TCP provides reliable, ordered, and error-checked delivery of a stream of octets (bytes) between applications running on hosts communicating via an IP network. Major internet applications such as the World Wide Web, email, remote administration, and file transfer rely on TCP, which is part of the Transport Layer of the TCP/IP suite. SSL/TLS often runs on top of TCP.

>传输控制协议（TCP）是互联网协议组中重要的组成部分之一。TCP的实现之初是为了补充互联网协议（IP）。因此，这一对组合经常被称为TCP/IP。TCP协议的特点：**可靠性**、**顺序性**、**错误检查**、**数据流传输**。大部分常见的应用都是使用的是TCP协议，例如：*World Wide Web*、*Email*、*文件传输*。

TCP is connection-oriented, and a connection between client and server is established (passive open) before data can be sent.
> TCP是面向连接的，这就需要客户端主动与服务端建立起连接之后才能开始传输数据。如果应用不需要可靠的数据流传输服务可能UPD协议是个更好的选择。

## History  

In May 1974, Vint Cerf and Bob Kahn described an internetworking protocol for sharing resources using packet switching among network nodes. The authors had been working with Gérard Le Lann  to incorporate concepts from the French CYCLADES project into the new network.The specification of the resulting protocol, RFC 675 (Specification of Internet Transmission Control Program), was written by Vint Cerf, Yogen Dalal, and Carl Sunshine, and published in December 1974. It contains the first attested use of the term Internet, as a shorthand for internetworking.

A central control component of this model was the Transmission Control Program that incorporated both connection-oriented links and datagram services between hosts. The monolithic Transmission Control Program was later divided into a modular architecture consisting of the Transmission Control Protocol and the Internet Protocol. This resulted in a networking model that became known informally as TCP/IP, although formally it was variously referred to as the Department of Defense (DOD) model, and ARPANET model, and eventually also as the Internet Protocol Suite.

In 2004, Vint Cerf and Bob Kahn received the Turing Award for their foundational work on TCP/IP.

## Why TCP ?

TCP is important because it establishes the rules and standard procedures for the way information is communicated over the internet. It is the foundation for the internet as it exists today and ensures that data transmission is carried out uniformly, regardless of the location, hardware or software involved. For this reason, it is flexible and highly scalable, meaning new protocols can be introduced to it and it will accommodate them. It is also nonproprietary, meaning no one person or company owns it.
>上面这段大概说的是：之所以TCP是个重要的协议是因为它在互联网的基础上建立了一套规则与标准化的信息数据传输机制。TCP让互联网之间的数据传输能够无视各种条件的限制（不管是地区限制、软件或硬件的限制），并且它是灵活的可拓展的（比如TCP/IP、增加SSL等），并且它是开源的不是私人所有。

## How TCP guarantee the reliability ?

### TCP segment structure

A TCP segment consists of a segment header and a data section. The segment header contains 10 mandatory fields, and an optional extension field (Options, pink background in table). The data section follows the header and is the payload data carried for the application. The length of the data section is not specified in the segment header; It can be calculated by subtracting the combined length of the segment header and IP header from the total IP datagram length specified in the IP header.
>TCP报文段分为两段：**报头**、**数据段**。*报头*包含10个字段和一个可选的拓展字段。*数据段*紧跟在 *报头*后面，里面装的都是应用传输的数据。数据的大小（length）不包括在 *报头*中；但是可以通过IP数据报的报头中指定的长度减去TCP报头的长度就得到了数据段的大小了。

![TCP structure](tcp%20structure.png)

#### Source Port(16 bits) & Destination Port(16 bits)

Identifies the sending port and Identifies the receiving port.

#### Sequence number(32 bits)

>序列号就是用来标记每一个请求的，相当于请求的ID

- If the SYN flag is set (1), then this is the initial sequence number. The sequence number of the actual first data byte and the acknowledged number in the corresponding ACK are then this sequence number plus 1.
> 如果含有同步标识（SYN），则此为最初的序列号；第一个数据比特的序列码为本序列号加一
- If the SYN flag is clear (0), then this is the accumulated sequence number of the first data byte of this segment for the current session.
>如果没有同步标识（SYN），则此为第一个数据比特的序列码

#### Acknowledgment number (32 bits)

If the ACK flag is set then the value of this field is the next sequence number that the sender of the ACK is expecting. 
>ACK如果被启用了，那么表示收到信息，并且ACK=（发送者的序列号+1）用于验证。

#### Flags (9 bits)

- NS (1 bit): ECN-nonce - concealment protection
- CWR (1 bit): Congestion window reduced (CWR) flag is set by the sending host to indicate that it received a TCP segment with the 
- ECE flag set and had responded in congestion control mechanism.
- ECE (1 bit): ECN-Echo has a dual role, depending on the value of the SYN flag. It indicates:
  - If the SYN flag is set (1), that the TCP peer is ECN capable.
  - If the SYN flag is clear (0), that a packet with Congestion Experienced flag set (ECN=11) in the IP header was received during normal transmission.This serves as an indication of network congestion (or impending congestion) to the TCP sender.
- URG (1 bit): Indicates that the Urgent pointer field is significant
- ACK (1 bit): Indicates that the Acknowledgment field is significant. All packets after the initial SYN packet sent by the client should have this flag set.
- PSH (1 bit): Push function. Asks to push the buffered data to the receiving application.
- RST (1 bit): Reset the connection
- SYN (1 bit): Synchronize sequence numbers. Only the first packet sent from each end should have this flag set. Some other flags and fields change meaning based on this flag, and some are only valid when it is set, and others when it is clear.
- FIN (1 bit): Last packet from sender

### Protocol operation

TCP protocol operations may be divided into three phases. Connections must be properly established in a multi-step handshake process (connection establishment) before entering the data transfer phase. After data transmission is completed, the connection termination closes established virtual circuits and releases all allocated resources.

A TCP connection is managed by an operating system through a resource that represents the local end-point for communications, the Internet socket. During the lifetime of a TCP connection, the local end-point undergoes a series of state changes:

#### LISTEN
(server) represents waiting for a connection request from any remote TCP and port.
#### SYN-SENT
(client) represents waiting for a matching connection request after having sent a connection request.
#### SYN-RECEIVED
(server) represents waiting for a confirming connection request acknowledgment after having both received and sent a connection request.
#### ESTABLISHED
(both server and client) represents an open connection, data received can be delivered to the user. The normal state for the data transfer phase of the connection.
#### FIN-WAIT-1
(both server and client) represents waiting for a connection termination request from the remote TCP, or an acknowledgment of the connection termination request previously sent.
#### FIN-WAIT-2
(both server and client) represents waiting for a connection termination request from the remote TCP.
#### CLOSE-WAIT
(both server and client) represents waiting for a connection termination request from the local user.
#### CLOSING
(both server and client) represents waiting for a connection termination request acknowledgment from the remote TCP.
#### LAST-ACK
(both server and client) represents waiting for an acknowledgment of the connection termination request previously sent to the remote TCP (which includes an acknowledgment of its connection termination request).
#### TIME-WAIT
(either server or client) represents waiting for enough time to pass to be sure the remote TCP received the acknowledgment of its connection termination request. [According to RFC 793 a connection can stay in TIME-WAIT for a maximum of four minutes known as two maximum segment lifetime (MSL).]
#### CLOSED
(both server and client) represents no connection state at all.

#### Connection establishment

To establish a connection, TCP uses a three-way handshake. Before a client attempts to connect with a server, the server must first bind to and listen at a port to open it up for connections: this is called a passive open. Once the passive open is established, a client may initiate an active open. To establish a connection, the three-way (or 3-step) handshake occurs:
>TCP连接的建立需要进行**三次握手**。在客户端尝试连接服务端之前，服务端需要绑定一个端口并监听它（passive open），之后客户端会主动向服务端发送建立连接请求，并进行 *三次握手* 建立连接。

SYN: The active open is performed by the client sending a SYN to the server. The client sets the segment's sequence number to a random value A.
>客户端向服务端发起一个 **SYN**请求，并将生成一个随机的序列号A。

SYN-ACK: In response, the server replies with a SYN-ACK. The acknowledgment number is set to one more than the received sequence number i.e. A+1, and the sequence number that the server chooses for the packet is another random number, B.
>作为响应，服务端回复带有SYN-ACK标示的应答。其中ACK=A+1，并同时生成一个随机序列号B。

ACK: Finally, the client sends an ACK back to the server. The sequence number is set to the received acknowledgement value i.e. A+1, and the acknowledgement number is set to one more than the received sequence number i.e. B+1.
>最后，客户端收到服务端的ACK应答之后。客户端又会给服务端一个ACK应答并且序列号也会加一也A+1，然后ACK的序列号也加一B+1

At this point, both the client and server have received an acknowledgment of the connection. The steps 1, 2 establish the connection parameter (sequence number) for one direction and it is acknowledged. The steps 2, 3 establish the connection parameter (sequence number) for the other direction and it is acknowledged. With these, a full-duplex communication is established.
>到这时候，客户端和服务端都收到了确认连接的消息。步骤1、2建立一个方向的连接参数(序列号)并确认。步骤2、3建立另一个方向的连接参数(序列号)并确认。有了这些，就可以建立全双工通信。

![establish](establish.png)

#### Connection termination

The connection termination phase uses a four-way handshake, with each side of the connection terminating independently. When an endpoint wishes to stop its half of the connection, it transmits a FIN packet, which the other end acknowledges with an ACK. Therefore, a typical tear-down requires a pair of FIN and ACK segments from each TCP endpoint. After the side that sent the first FIN has responded with the final ACK, it waits for a timeout before finally closing the connection, during which time the local port is unavailable for new connections; this prevents confusion due to delayed packets being delivered during subsequent connections.

>TCP连接的关闭需要进行 **四次挥手**，两边都需要关闭连接，整个TCP连接才算完整的关闭。希望关闭连接，就会传送 **FIN** 包并包含之前的ACK包。所以关闭连接需要 一组 **FIN** 和 **ACK**。当一边的发送了第一个 **FIN**和最后的 **ACK**，会进入一个等待超时状态 **TIME_WATI** 之道整个连接关闭，在整个连接关闭之前TCP占用的端口不能被其他新连接所使用；这防止了由于延迟发包带来的混乱问题。

![termination](termination.png)

A connection can be "half-open", in which case one side has terminated its end, but the other has not. The side that has terminated can no longer send any data into the connection, but the other side can. The terminating side should continue reading the data until the other side terminates as well.
>连接可以处于**半开**状态，意思就是TCP的有一方连接已经关闭了，另一方还没有关闭。

![half-open](half&#32;open.png)

### Data transfer

#### Reliable transmission
TCP uses a sequence number to identify each byte of data. The sequence number identifies the order of the bytes sent from each computer so that the data can be reconstructed in order, regardless of any packet reordering, or packet loss that may occur during transmission. The sequence number of the first byte is chosen by the transmitter for the first packet, which is flagged SYN. This number can be arbitrary, and should, in fact, be unpredictable to defend against TCP sequence prediction attacks.
>TCP使用 **序列号**标记每一份数据，通过使用序号和确认号，TCP 层可以把收到的报文段中的字节按正确的顺序交付给应用层，TCP 协议使用序号标识每端发出的字节的顺序，从而另一端接收数据时可以重建顺序，无惧传输时的包的乱序交付或丢包。在发送第一个包时（SYN包），选择一个 随机数 作为序号的初值，以克制 TCP 序号预测攻击。

Acknowledgements (ACKs) are sent with a sequence number by the receiver of data to tell the sender that data has been received to the specified byte. ACKs do not imply that the data has been delivered to the application. They merely signify that it is now the receiver's responsibility to deliver the data.
>发送确认包（Acks），携带了接收到的对方发来的字节流的编号，称为确认号，以告诉对方 已经成功接收的数据流的字节位置。Ack并不意味着数据已经交付了上层应用程序。可

Reliability is achieved by the sender detecting lost data and retransmitting it. TCP uses two primary techniques to identify loss. Retransmission timeout (abbreviated as RTO) and duplicate cumulative acknowledgements (DupAcks).
>靠性通过发送方检测到丢失的传输数据并重传这些数据。包括 超时重传（Retransmission timeout，RTO）与 重复累计确认 （duplicate cumulative acknowledgements，DupAcks）。

#### Dupack-based retransmission

If a single segment (say segment 100) in a stream is lost, then the receiver cannot acknowledge packets above no. 100 because it uses cumulative ACKs. Hence the receiver acknowledges packet 99 again on the receipt of another data packet. This duplicate acknowledgement is used as a signal for packet loss. That is, if the sender receives three duplicate acknowledgements, it retransmits the last unacknowledged packet. A threshold of three is used because the network may reorder segments causing duplicate acknowledgements. This threshold has been demonstrated to avoid spurious retransmissions due to reordering. Sometimes selective acknowledgements (SACKs) are used to provide explicit feedback about the segments that have been received. This greatly improves TCP's ability to retransmit the right segments.
>如果一个包（不妨设它的序号是 100 ，即该包始于第 100 字节）丢失，接收方就不能确认这个包及其以后的包，因为采用了 累计ACK 。接收方在收到 100 以后的包时，发出对包含第 99 字节的包的确认。这种重复确认是包丢失的信号。发送方如果收到 3 次对同一个包的确认，就重传最后一个未被确认的包。阈值设为 3 被证实可以减少乱序包导致的无作用的重传（spurious retransmission）现象。**选择性确认（SACK）**的使用能明确反馈哪个包收到了，极大改善了TCP重传必要的包的能力。

#### Timeout-based retransmission

When a sender transmits a segment, it initializes a timer with a conservative estimate of the arrival time of the acknowledgement. The segment is retransmitted if the timer expires prematurely, with a new timeout threshold of twice the previous value, resulting in exponential backoff behavior. Typically, the initial timer value is {{<katex>}}{\text{smoothed RTT}}+\max(G,4\times {\text{RTT variation}}){{</katex>}}, where {\displaystyle G}G is the clock granularity.This guards against excessive transmission traffic due to faulty or malicious actors, such as man-in-the-middle denial of service attackers.
>发送方使用一个保守估计的时间作为收到数据包的确认的超时上限。如果超过这个上限仍未收到确认包，发送方将重传这个数据包。每当发送方收到确认包后，会重置这个重传定时器。典型地，定时器的值设定为 {{<katex>}}{\text{smoothed RTT}}+\max(G,4\times {\text{RTT variation}}){{</katex>}} 其中 {{<katex>}}G{{</katex>}} 是时钟粒度。进一步，如果重传定时器被触发，仍然没有收到确认包，定时器的值将被设为前次值的二倍（直到特定阈值）。这可对抗 中间人攻击方式的拒绝服务攻击，这种攻击愚弄发送者重传很多次导致接受者被压垮。

## When we need TCP ?

## Where we using TCP ?

## Conclusion

## Reference

 - [TCP Protocol From Wikipedia](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#Historical_origin)
 - [Why TCP is important](https://searchnetworking.techtarget.com/definition/TCP)
 - 《TCP IP 详解 第四版》