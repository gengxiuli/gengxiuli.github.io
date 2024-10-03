---
layout: post
title:  "谈谈 ssh 的那些事儿"
date:   2024-10-03
category: networking
tags:   ssh
---

最近看到一篇文章[那些 ssh 教我的事](https://yangwenbo.com/articles/ssh-oh-my-god.html)，虽然是 2009 年发布的，虽然我也用了 ssh 很多年，但这些事儿我还真不知道，比如 ssh -L 和 ssh -R 这两个选项的作用，即使 sftp 知道也用过，但是没有具体去研究原理，所以借着国庆休假的时间，谈谈 ssh 的那些事儿。