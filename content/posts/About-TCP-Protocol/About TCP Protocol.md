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

TCP is connection-oriented, and a connection between client and server is established (passive open) before data can be sent. Three-way handshake (active open), retransmission, and error-detection adds to reliability but lengthens latency. Applications that do not require reliable data stream service may use the User Datagram Protocol (UDP), which provides a connectionless datagram service that prioritizes time over reliability. TCP employs network congestion avoidance. However, there are vulnerabilities to TCP including denial of service, connection hijacking, TCP veto, and reset attack. For network security, monitoring, and debugging, TCP traffic can be intercepted and logged with a packet sniffer.

## Who create TCP Protocol ?

In May 1974, Vint Cerf and Bob Kahn described an internetworking protocol for sharing resources using packet switching among network nodes. The authors had been working with Gérard Le Lann  to incorporate concepts from the French CYCLADES project into the new network.The specification of the resulting protocol, RFC 675 (Specification of Internet Transmission Control Program), was written by Vint Cerf, Yogen Dalal, and Carl Sunshine, and published in December 1974. It contains the first attested use of the term Internet, as a shorthand for internetworking.

A central control component of this model was the Transmission Control Program that incorporated both connection-oriented links and datagram services between hosts. The monolithic Transmission Control Program was later divided into a modular architecture consisting of the Transmission Control Protocol and the Internet Protocol. This resulted in a networking model that became known informally as TCP/IP, although formally it was variously referred to as the Department of Defense (DOD) model, and ARPANET model, and eventually also as the Internet Protocol Suite.

In 2004, Vint Cerf and Bob Kahn received the Turing Award for their foundational work on TCP/IP.

## Why TCP ?

For realiable and send big data package

## How TCP guarantee the reliability ?

## When we need TCP ?

## Where we using TCP ?

## Conclusion

## Reference

 - [TCP Protocol From Wikipedia](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#Historical_origin)