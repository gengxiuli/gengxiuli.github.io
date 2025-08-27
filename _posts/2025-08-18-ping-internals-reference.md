---
layout: post
title:  "Ping internals reference"
date:   2025-08-18
category: networking
tags:   linux kernel ping
---

ping在网络领域是一个十分常用的工具，经常用来检测网络可达或者网络延迟，几乎所有具有网络功能的操作系统都支持。

ping在实现上主要使用了RFC792中定义的ICMP协议，具体说是使用了其中的echo requst和echo reply两种报文。

在GNU/Linux环境中，ping实现在iputils工具包中，相对于Windows下的ping功能更加强大，提供了丰富的配置参数。

## Reference
1. [https://wiki.linuxfoundation.org/networking/iputils](https://wiki.linuxfoundation.org/networking/iputils)
2. [https://github.com/iputils/iputils/](https://github.com/iputils/iputils/)
3. [https://git.launchpad.net/ubuntu/+source/iputils/tree/ping?h=ubuntu/plucky](https://git.launchpad.net/ubuntu/+source/iputils/tree/ping?h=ubuntu/plucky)
4. [https://tracker.debian.org/pkg/iputils](https://tracker.debian.org/pkg/iputils)
5. [https://src.fedoraproject.org/rpms/iputils](https://src.fedoraproject.org/rpms/iputils)
6. [https://elixir.bootlin.com/busybox/1.37.0/source/networking/ping.c](https://elixir.bootlin.com/busybox/1.37.0/source/networking/ping.c)
7. [https://www.rfc-editor.org/rfc/rfc792](https://www.rfc-editor.org/rfc/rfc792)

