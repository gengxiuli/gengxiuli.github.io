---
layout: post
title:  "谈谈 SSL 的那些事儿"
date:   2024-10-05
category: networking
tags:   SSL https
---

上一篇文章谈了谈关于 ssh 的那些事儿，对于系统性理解 ssh 有指导作用，这一篇谈一谈 SSL 的那些事儿，二者有类似也有差异的地方，看完两篇文章后，这些异同点自然就体现了出来。另外注意本文提到 SSL 的时候都是大写形式，因为这是一项技术，而不是某个工具或者命令。

其实最近刚刚使用了 Let's Encropyt 在网站上部署了 HTTPS 服务，而 HTTPS 依赖的正是 SSL/TLS 技术，这里的 TLS 是比 SSL 更新的标准，可以认为是 SSL 的继承者，TLS 技术标准工作由 IETF 中的The TLS (Transport Layer Security) working group 负责完成。在[工作组介绍页面](https://datatracker.ietf.org/wg/tls/about)中提及了 SSL 和 TLS 的相关 RFC:

> The basis for the work was SSL (Secure Socket Layer) v3.0 [RFC6101]. The TLS working group has completed a series of specifications that describe the TLS protocol v1.0 [RFC2246], v1.1 [RFC4346], v1.2 [RFC5246], and v1.3 [RFC8446], and DTLS (Datagram TLS) v1.0 [RFC4347], v1.2 [RFC6347], and v1.3 [draft-ietf-tls-dtls13], as well as extensions to the protocols and ciphersuites.

那么 HTTPS 和 SSL/TLS 是什么关系呢？其实 HTTPS 是 HTTP 协议从 HTTP/2 开始强制支持 TLS 协议的一种叫法，具体可以参考[RFC 9113 HTTP/2](https://www.rfc-editor.org/rfc/rfc9113.html)中的[3.2. Starting HTTP/2 for "https" URIs](https://www.rfc-editor.org/rfc/rfc9113.html#discover-https)和[9.2. Use of TLS Features](https://www.rfc-editor.org/rfc/rfc9113.html#name-use-of-tls-features)章节，最明显的变化就是浏览器中的URI(Uniform Resource Identifier)统一资源标识从http变为了https，反过来说要想利用HTTP/2以上协议的特性，客户端和服务器必须都支持HTTPS。除了该RFC标准，其他HTTP协议都由IETF的[HTTPWG](https://httpwg.org/)负责制定完成。

相对于于SSH来说，SSL涉及更加广泛使用的Web服务，而其中最重要的就是HTTP协议。所以SSL/TLS不仅仅在协议层提供了安全机制，而且还提供了证书授权机构CA(Certificate authority)机制。