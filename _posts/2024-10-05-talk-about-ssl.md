---
layout: post
title:  "谈谈 SSL 的那些事儿"
date:   2024-10-05
category: networking
tags:  ssl tls ssh http https nginx apache openssl
---

上一篇文章谈了谈关于 SSH 的那些事儿，对于系统性理解 SSH 有指导作用，这一篇谈一谈 SSL 的那些事儿，二者有类似也有差异的地方，看完两篇文章后，这些异同点自然就体现了出来。另外注意本文提到 SSL 的时候都是大写形式，因为这是一项技术，而不是某个工具或者命令。

其实最近刚刚使用了 Let's Encropyt 在网站上部署了 HTTPS 服务，参考的文章是[免费 HTTPS 证书-从 StartSSL 到 Let's Encrypt](https://yangwenbo.com/articles/free-https-cert-startssl-to-letsencrypt.html)。而 HTTPS 依赖的正是 SSL/TLS 技术，这里的 TLS 是比 SSL 更新的标准，可以认为是 SSL 的继承者，TLS 技术标准工作由 IETF 中的The TLS (Transport Layer Security) working group 负责完成。在[工作组介绍页面](https://datatracker.ietf.org/wg/tls/about)中提及了 SSL 和 TLS 的相关 RFC:

> The basis for the work was SSL (Secure Socket Layer) v3.0 [RFC6101]. The TLS working group has completed a series of specifications that describe the TLS protocol v1.0 [RFC2246], v1.1 [RFC4346], v1.2 [RFC5246], and v1.3 [RFC8446], and DTLS (Datagram TLS) v1.0 [RFC4347], v1.2 [RFC6347], and v1.3 [draft-ietf-tls-dtls13], as well as extensions to the protocols and ciphersuites.

那么 HTTPS 和 SSL/TLS 是什么关系呢？其实 HTTPS 是 HTTP 协议从 HTTP/2 开始强制支持 TLS 协议的一种叫法，具体可以参考[RFC 9113 HTTP/2](https://www.rfc-editor.org/rfc/rfc9113.html)中的[3.2. Starting HTTP/2 for "https" URIs](https://www.rfc-editor.org/rfc/rfc9113.html#discover-https)和[9.2. Use of TLS Features](https://www.rfc-editor.org/rfc/rfc9113.html#name-use-of-tls-features)章节，最明显的变化就是浏览器中的URI(Uniform Resource Identifier)统一资源标识从http变为了https，反过来说要想利用HTTP/2以上协议的特性，客户端和服务器必须都支持HTTPS。除了该RFC标准，其他HTTP协议都由IETF的[HTTPWG](https://httpwg.org/)负责制定完成。

相对于于SSH来说，SSL涉及更加广泛使用的Web服务，而其中最重要的就是HTTP协议。所以SSL/TLS不仅仅在协议层提供了安全机制，而且还提供了证书授权机构CA(Certificate authority)机制。简单来说，证书授权机构针对的是非对称加密中公钥的有效性进行公开签发和授权，这样其他使用者就可以信任对这些拥有证书的网站的访问。这样可以防止中间人篡改证书而发起的中间人攻击，其实SSH在协议[The Secure Shell (SSH) Protocol Architecture](https://datatracker.ietf.org/doc/html/rfc4251#section-4.1)中也提及了这种机制。注意这里是对网站证书的授权认证，而不是网站的授权认证。

目前绝大多数浏览器和操作系统都集成了受信任的权威证书授权机构名单，Wiki上的统计数据如下：

> As of 24 August 2020, 147 root certificates, representing 52 organizations, are trusted in the Mozilla Firefox web browser, 168 root certificates, representing 60 organizations, are trusted by macOS, and 255 root certificates, representing 101 organizations, are trusted by Microsoft Windows. As of Android 4.2 (Jelly Bean), Android currently contains over 100 CAs that are updated with each release.

SSL/TLS最被人熟知的实现就是[OpenSSL](https://www.openssl.org/)，由于几乎所有需要使用HTTPS服务的客户端和服务器都需要SSL/TLS功能，所以OpenSSL也被大量广泛的使用。Google的Chrome和Microsoft的Edge浏览器也使用了OpenSSL，但是他们使用fork+patch的方式创建了一个项目[boringssl](https://github.com/google/Boringssl)，其中项目介绍中写到：

> BoringSSL is a fork of OpenSSL that is designed to meet Google's needs.

> Although BoringSSL is an open source project, it is not intended for general use, as OpenSSL is. We don't recommend that third parties depend upon it. Doing so is likely to be frustrating because there are no guarantees of API or ABI stability.

也就是说，这个项目是Google为了自己的产品需要而创建的，不能保证对于第三方使用的API或ABI兼容。Google还解释了具体的细节：

> BoringSSL arose because Google used OpenSSL for many years in various ways and, over time, built up a large number of patches that were maintained while tracking upstream OpenSSL. As Google's product portfolio became more complex, more copies of OpenSSL sprung up and the effort involved in maintaining all these patches in multiple places was growing steadily.

总结起来就是这样自己使用起来很灵活，既可以很快根据特定需要作出修改，不用考虑合入OpenSSL主线，还可以同步OpenSSL upstream的修改，及时借鉴开源的贡献。但这种方式需要很强的研发实力，也只有Google这种级别的公司可以做到。确实，这种使用方法看似操作简单，不过能做出Google下面几个产品这种级别的，而且能开源出来BoringSSL的却不多。

> Currently BoringSSL is the SSL library in Chrome/Chromium, Android (but it's not part of the NDK) and a number of other apps/programs.

除了上面的客户端浏览器之外，开源Web服务器Nginx和Apache都支持ssl模块，而这些模块背后使用的都是OpenSSL。

Nginx [Module ngx_http_ssl_module](https://nginx.org/en/docs/http/ngx_http_ssl_module.html)

> The ngx_http_ssl_module module provides the necessary support for HTTPS.
This module is not built by default, it should be enabled with the --with-http_ssl_module configuration parameter.
This module requires the **OpenSSL** library.

Apache [Apache SSL/TLS Encryption](https://httpd.apache.org/docs/2.4/ssl/)

> The Apache HTTP Server module mod_ssl provides an interface to the **OpenSSL** library, which provides Strong Encryption using the Secure Sockets Layer and Transport Layer Security protocols.

OpenSSL 在 2014 年 4 月 还出现过一次影响比较大的漏洞，具体是其在 2012 年 12 月引入的heartbeat 扩展功能，其代码中存在缓冲区溢出 bug，可能导致服务器被恶意程序攻击，具体可以参考 Wiki 页面[Heartbleed](https://en.m.wikipedia.org/wiki/Heartbleed)。这次影响范围之大还体现在，专门有一个关于这个问题的网站[https://heartbleed.com](https://heartbleed.com)。这个网站详细解释了这个 bug 的影响，有兴趣可以移步浏览，其中需要注意的是，这个 bug 是 OpenSSL 实现中引入的问题，而不是 SSL/TLS 加密机制本身设计的问题。

其他参考资料
1. [What is SSL, TLS & HTTPS?](https://www.digicert.com/what-is-ssl-tls-and-https)
2. [What is SSL? SSL definition](https://www.cloudflare.com/learning/ssl/what-is-ssl)
3. [What is TLS (Transport Layer Security)?](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/)
4. [What is SSL/TLS: An In-Depth Guide](https://www.ssl.com/article/what-is-ssl-tls-an-in-depth-guide)
5. [What is an SSL certificate – Definition and Explanation](https://www.kaspersky.com/resource-center/definitions/what-is-a-ssl-certificate)