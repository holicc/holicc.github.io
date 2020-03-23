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

### Connection establishment

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

### Connection termination

The connection termination phase uses a four-way handshake, with each side of the connection terminating independently. When an endpoint wishes to stop its half of the connection, it transmits a FIN packet, which the other end acknowledges with an ACK. Therefore, a typical tear-down requires a pair of FIN and ACK segments from each TCP endpoint. After the side that sent the first FIN has responded with the final ACK, it waits for a timeout before finally closing the connection, during which time the local port is unavailable for new connections; this prevents confusion due to delayed packets being delivered during subsequent connections.

A connection can be "half-open", in which case one side has terminated its end, but the other has not. The side that has terminated can no longer send any data into the connection, but the other side can. The terminating side should continue reading the data until the other side terminates as well.

It is also possible to terminate the connection by a 3-way handshake, when host A sends a FIN and host B replies with a FIN & ACK (merely combines 2 steps into one) and host A replies with an ACK.

Some host TCP stacks may implement a half-duplex close sequence, as Linux or HP-UX do. If such a host actively closes a connection but still has not read all the incoming data the stack already received from the link, this host sends a RST (losing the received data) instead of a FIN (Section 4.2.2.13 in RFC 1122). This allows a TCP application to be sure the remote application has read all the data the former sent—waiting the FIN from the remote side, when it actively closes the connection. But the remote TCP stack cannot distinguish between a Connection Aborting RST and Data Loss RST. Both cause the remote stack to lose all the data received.

### Protocol operation



## When we need TCP ?

## Where we using TCP ?

## Conclusion

## Reference

 - [TCP Protocol From Wikipedia](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#Historical_origin)
 - [Why TCP is important](https://searchnetworking.techtarget.com/definition/TCP)
 - 《TCP IP 详解 第四版》