---
layout: post
title:  "Protocol Ossification"
date:   2026-03-17
category: networking
tags:   tcp bgp quic
---

最近在 APNIC blog 上看到了一篇文章[Ossification and the Internet](https://blog.apnic.net/2025/06/25/ossification-and-the-internet/)，里面提到了 ossification 这个词。ossification 可以翻译为骨化或者石化，对于网络来说，就是意味着成熟且广泛使用的协议，很难去改进和进化，以至于网络长期停留在某一个特定阶段。文中列举了现在使用的一些主要协议，包括网络层的 IP，传输层的 TCP 和 UDP，路由协议中的 BGP，应用层的 DNS， 还有网络层的分片机制和最大包长。

[QUIC as a solution to protocol ossification](https://lwn.net/Articles/745590/)这篇文章分析了 QUIC 协议是如何在 HTTP 传输领域一步步替代 TCP 的，使用 UDP 作为底层也是迫不得已。从另一个角度说，既然很难改变 TCP，那就另起炉灶一个新的协议，也是一种很好的思路。

[Checksum offloads and protocol ossification](https://lwn.net/Articles/667059/)这篇文章分析了协议校验和卸载和协议骨化的关系。