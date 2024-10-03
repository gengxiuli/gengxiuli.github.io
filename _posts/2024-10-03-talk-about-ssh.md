---
layout: post
title:  "谈谈 ssh 的那些事儿"
date:   2024-10-03
category: networking
tags:   ssh
---

最近看到一篇文章[那些 ssh 教我的事](https://yangwenbo.com/articles/ssh-oh-my-god.html)，虽然是 2009 年发布的，虽然我也用了 ssh 很多年，但这些事儿我还真不知道，比如 ssh -L 和 ssh -R 这两个选项的作用，即使 sftp 知道也用过，但是没有具体去研究原理，所以借着国庆休假的时间，谈谈 ssh 的那些事儿。

ssh 最广为人知的用处就是远程登录了，它比 telnet 更安全，因此现在绝大部分网络设备和服务器都支持 ssh 服务器，允许通过 ssh 客户端登录管理。我们使用这种方式登录服务器的时候，一般都会看到一些提示窗口，询问我们是否保存服务器密钥指纹(key fingerprint)，确认之后我们才能够使用用户名和密码登录服务器。[什么是SSH？](https://info.support.huawei.com/info-finder/encyclopedia/zh/SSH.html)这篇文章详细介绍了 ssh 的工作原理，这里就不再一一赘述了。需要注意的几点是：
