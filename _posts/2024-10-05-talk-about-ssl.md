---
layout: post
title:  "谈谈 SSL 的那些事儿"
date:   2024-10-05
category: networking
tags:   SSL https
---

上一篇文章谈了谈关于 ssh 的那些事儿，对于系统性理解 ssh 有指导作用，这一篇谈一谈 SSL 的那些事儿，二者有类似也有差异的地方，看完两篇文章后，这些异同点自然就体现了出来。另外注意本文提到 SSL 的时候都是大写形式，因为这是一项技术，而不是某个工具或者命令。

其实最近刚刚使用了 Let's Encropyt 在网站上部署了 HTTPS 服务，而 HTTPS 依赖的正是 SSL/TLS 技术，这里的 TLS 是比 SSL 更新的标准，可以认为是 SSL 的继承者，TLS 技术标准工作由 IETF 中的The TLS (Transport Layer Security) working group 负责完成。