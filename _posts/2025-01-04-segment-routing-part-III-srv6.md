---
layout: post
title:  "Segment Routing Part III:SRv6"
date:   2025-01-04
category: networking
tags:   srv6
---

[Segment Routing Part III: SRv6](https://www.amazon.com/Segment-Routing-Part-III-SRv6-ebook/dp/B0D6GWWRWH)英文版现在已经出版，Kinle版本可以购买，Amazon上的出版日期显示为：October 12, 2024。估计中文纸质版应该也不会等太久。
[]
从标题也可以看出，这个Part III的关注点就是Segment Routing over IPv6 (SRv6)。SRv6是这几年网络领域比较热门的技术了，国外和国内都有研究。

[RFC 8754 IPv6 Segment Routing Header (SRH)](https://www.rfc-editor.org/rfc/rfc8754.html)和[RFC 8986
Segment Routing over IPv6 (SRv6) Network Programming](https://www.rfc-editor.org/rfc/rfc8986)是比较早的IETF标准，之后有相关的OSPF/IS-IS/BGP扩展，以及OAM相关机制。

在国内各大运营商的汇聚和核心路由器早已对SRv6有技术要求，但是在终端方面还没有大规模部署，这里面有很多因素在制约。在国外很多设备商和运营商也在积极推进SRv6的演进，但相对于成熟的MPLS或者SR MPLS，还是存在一些技术问题需要解决。在这其中，对于SRv6的压缩算法，在标准化方面还存在难以统一的现状，相信还需要有很长的路要走。

下面是英文版的简介：

> Unleash the Full Potential of Your Network with SRv6
In the ever-evolving landscape of network engineering, staying ahead of the curve is not just an advantage; it's a necessity. This book, the third installment of the Segment Routing series, is the definitive textbook for engineers and IT professionals ready to master the latest revolution in networking technology: SRv6.
> 
> SRv6 emerges as the key enabler of a new era of networking. Integrating the Segment Routing functionalities with the native capabilities of IPv6 to create a self-sufficient solution. This powerful synergy unlocks possibilities for increased network efficiency, enhanced control, and greater versatility, which are critical in modern networking environments.
> 
> The SRv6 integration enables “IPv6 for Any Service Anywhere”. Eliminating the additional protocol layers (MPLS, VxLAN, NSH) allows for building any service (TDM, VPN, slicing, Traffic Engineering, Green routing, FRR, NFV, etc.) end-to-end, across any domain (Access, Metro, Core, DC, Cloud, Host). While offering a universal solution, SRv6 outperforms all the current market-specific solutions across the board.
> 
> Hardware optimizations ensure line-rate performance across various hardware platforms. By leveraging the highly optimized longest-prefix match, network programs are embedded in the IPv6 destination address, providing ultimate simplicity for engineering service traffic.
> 
> This does not result in huge packet headers; on the contrary, SRv6 uSIDs provide the smallest MTU overhead. Only the base IPv6 header is required to engineer packet flows through the stateless network fabric; the use of a Segment Routing Header (SRH) is seldom necessary.
> 
> As a backward-compatible integrated IPv6 solution, SRv6 is seamlessly deployable in brownfield networks.
> 
> SRv6 is an ultra-scalable solution that enhances both network and service scalability. Owing to the simplified network stack, prefix summarization is feasible again, while maintaining end-to-end Traffic Engineering capability.
> 
> SRv6 is not only simple, performant, and scalable; it is versatile as well. With the network programming paradigm, it provides flexible design options suitable for a wide array of network deployments, from defense to enterprise to service providers to hyperscalers.
> 
> The SRv6 solution is standardized with wide industry support and a rich eco-system including a rich open-source community.
> 
> SRv6 establishes a solid foundation for the continuous growth and integration of next-generation network innovations. This positions networks to be agile, scalable, and ready for the burgeoning demands of future data transmission and communication needs.
> 
> In this book, you will discover:
> 
> An introduction to SRv6, defining its role in the future of networking.
Detailed explanations of the SRv6 architecture, including its components and how it integrates with and enhances IPv6 networks.
Clear explanations of SRv6 operations, including network programming, packet forwarding mechanics, and service chaining.
Insights into SRv6 deployment scenarios, enabling services to use SRv6 transport and steering over user-defined algorithmic paths leveraging Flex-Algo.
Showcasing how SRv6 provides network resiliency using TI-LFA, micro-loop avoidance, and BGP Prefix-Independent Convergence (PIC).
Best practices for implementing and optimizing SRv6, ensuring you can maximize performance and reliability.
Case studies of real-world SRv6 deployments.
> 
> While you can read this book independently from the previous books in the series, it assumes a basic understanding of Segment Routing functionalities.

