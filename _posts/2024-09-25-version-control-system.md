---
layout: post
title:  version control system
date:   2024-09-25
category: networking 
tags:   git subversion  
---

版本控制系统

Subversion Mercurial Git

[为什么我们要放弃 Subversion](https://www.infoq.cn/article/thoughtworks-practice-partiv/)
作者介绍了从Subversion迁移到Mercurial的过程，因为当时git还没这么流行，事实上github成立于2008年。因此作者那时候(2009年)选择了
Mercurial，从功能看其实二者有很多相似之处。另外，作者的[博客](https://iamhukai.blogspot.com)还有一些关于Mercurial的使用总结。

云风的这篇[分布式的版本控制工具](https://blog.codingnow.com/2008/01/distributed_version_control.html)，发表于2008年，也指出了Subversion在使用中的一些优缺点，同时分析了分布式版本控制系统的一些可能优势，另外提出了一些使用场景。因为受限于公司政策，svn的分支功能不方便使用，确实带来很多痛点。另外，作者当时提到git在Windows的支持情况，在现在已经得到了很大的改善，比如官方推荐的[gitforwindows](https://gitforwindows.org/)，其他Windows下GUI Clients工具可以参考[这里](https://git-scm.com/download/guis?os=windows)。