---
layout: post
title:  "理解nginx反向代理reverse proxy"
date:   2024-09-15
category: networking 
tags:   nginx proxy dpuio
---

在jia.je的站点搜索git相关文章，又找到了前不久看到的一篇关于nginx反向代理的文章。其实反向代理这个概念也遇到了好几次，没有怎么理解清楚，这次趁着有空，简单的梳理了一下相关概念。另外，在我自己的wordpress站点上，nginx只是作为web server的角色，因为站点小，反向代理用不上，但也不排除以后部署上。

# 基本原理

在nginx官网上[NGINX Reverse Proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)这篇文章里是这么介绍的：

> Proxying is typically used to distribute the load among several servers, seamlessly show content from different websites, or pass requests for processing to application servers over protocols other than HTTP.

简单说代理就是讲请求负载交给多个应用服务器，和该服务器交互可以是除了HTTP外的其他协议。这里其实涉及到一个设计思想，其在很多网络系统或者软件系统都有应用，那就是通过增加一个shim(垫层)，来实现更加灵活的功能，这里其实就代理服务器，他解耦了client和Web Server，虽然看似提高了复杂度，但是这样可以带来很多好处。

然后，下面介绍如何将请求交给被代理的服务器，其中说到：

> When NGINX proxies a request, it sends the request to a specified proxied server, fetches the response, and sends it back to the client. It is possible to proxy requests to an HTTP server (**another NGINX server or any other server**) or a non-HTTP server (which can run an application developed with a specific framework, such as PHP or Python) using a specified protocol. Supported protocols include [FastCGI](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html), [uwsgi](https://nginx.org/en/docs/http/ngx_http_uwsgi_module.html), [SCGI](https://nginx.org/en/docs/http/ngx_http_scgi_module.html), and [memcached](https://nginx.org/en/docs/http/ngx_http_memcached_module.html).其中关键的一点就是HTTP server可以是nginx，也可以是其他任何服务器，上文[Understanding Nginx As A Reverse Proxy](https://medium.com/globant/understanding-nginx-as-a-reverse-proxy-564f76e856b2)这篇文章就以ASP.NET服务端**Kestrel**为例，说明如何配置nginx作为反向代理服务器，另外，Microsoft官网ASP.NET Core中也有配置nginx作为反向代理的文章[Part 2.2 - Install Nginx and configure it as a reverse proxy server](https://learn.microsoft.com/en-us/troubleshoot/developer/webapps/aspnetcore/practice-troubleshoot-linux/2-2-install-nginx-configure-it-reverse-proxy),这也体现了反向代理的优势：可以灵活支持各种Web Server。

# 主要特点

[Understanding Nginx As A Reverse Proxy](https://medium.com/globant/understanding-nginx-as-a-reverse-proxy-564f76e856b2)这篇文章列了一些，比如负载均衡，安全性，缓存，日志，TLS以及各种协议支持，原文如下：

> **Load balancing**: Nginx load balance the client request to multiple upstream servers evenly which improve performance and provide redundancy in case of server failure. This helps to keep the application up all the time to serve client requests and provide better SLA for application.
> 
> **Security**: Nginx server provides security to backend servers that exist in the private network by hiding their identity. The backend servers are unknown to the client that are making requests. it also provides a single point of access to multiple backend servers regardless of the backend network topology.
> 
> **Caching**: Nginx can serve static content like image, videos, etc directly and deliver better performance. It reduces the load on the backend server by serving static content directly instead of forwarding it to the backend server for processing.
> 
> **Logging**: Nginx provides centralized logging for backed server request and response passing through it and provides a single place to audit and log for troubleshooting issues.
> 
> **TLS/SSL support**: Nginx allows secure communication between client and server using TLS/SSL connection. User data remains secure & encrypted while transferring over the wire using an HTTPS connection.
> 
> **Protocol Support**: Nginx supports HTTP, HTTPS, HTTP/1.1, HTTP/2, gRPC - Hypertext Transport Protocol along with both IP4 & IP6 internet protocol.

下图是nginx在Web应用中的位置，原文说是Image courtesy from NGINX，但是没有找到nginx的来源：

![nginx reverse proxy](/assets/img/2024/nginx-reverse-proxy.png)