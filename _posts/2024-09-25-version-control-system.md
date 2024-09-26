---
layout: post
title:  Version Control System
date:   2024-09-25
category: networking 
tags:   git subversion mercurial 
---

版本控制系统

Subversion Mercurial Git

[为什么我们要放弃 Subversion](https://www.infoq.cn/article/thoughtworks-practice-partiv/)
作者介绍了从Subversion迁移到Mercurial的过程，因为当时git还没这么流行，事实上github成立于2008年。因此作者那时候(2009年)选择了
Mercurial，从功能看其实二者有很多相似之处。另外，作者的[博客](https://iamhukai.blogspot.com)还有一些关于Mercurial的使用总结。

云风的这篇[分布式的版本控制工具](https://blog.codingnow.com/2008/01/distributed_version_control.html)，发表于2008年，也指出了Subversion在使用中的一些优缺点，同时分析了分布式版本控制系统的一些可能优势，另外提出了一些使用场景。因为受限于公司政策，svn的分支功能不方便使用，确实带来很多痛点。另外，作者当时提到git在Windows的支持情况，在现在已经得到了很大的改善，比如官方推荐的[gitforwindows](https://gitforwindows.org/)，其他Windows下GUI Clients工具可以参考[这里](https://git-scm.com/download/guis?os=windows)。

[在Google Code上用 Mercurial 取代 Subversion 管理你的项目](https://leeiio.me/googlecode-converting-svn-to-hg)这篇文章发表于2010年，请把链接地址中的https修改为http,主要关注在Google Code上用 Mercurial 取代 Subversion作为版本管理工具。我也是从这篇文章找到了上面旳两篇文章，其中还提到了其他几篇讨论版本控制工具的文章。大概也是2010年前后，我也在Google Code上托管过代码，类似Sourceforge提供版本控制、问题跟踪、Wiki、下载托管等工具。不过这个服务于2016年1月25日被Google彻底关闭了，所以本文的其他内容也无法参考了。

老万故事会上有篇文章[谷歌对微软：代码管理工具哪家强？](https://mp.weixin.qq.com/s/ckrH72rBp7_GT1UlfQsUaw)，发表于2023年，这几乎是git一统天下的时代。作者从工程师的角度分析和对比了谷歌和微软的版本工具使用情况，不是那种听说和猜测，而是亲身经历的第一手资料，也让我们看到世界级的科技巨头是如何管理代码仓库的。这里我觉得比较有价值的一点是，在解决内部痛点的时候，对于版本管理工具进步做出的贡献。比如git克隆了完整的历史提交进而占用大量本地资源，git的分支虽然快速灵活但是无法并行使用，git不方便只检出特定目录或者文件，虽然可以通过各种配置和命令解决或者规避，但是门槛太高或者代价堵太大，总之git有他适用的场景和不太适用的场景。谷歌用自己的方式解决了这些问题，同时也让我们思考集中式和分布式是否有合作演进的可能，毕竟我们目的是满足实际需求，而不是争论谁优谁劣。

比较知名的嵌入式设备开源SSH实现dropbear，在2022年还在使用Mercurial作为版本控制工具，站点在[这里](https://hg.ucc.asn.au/dropbear)。但是在2022年的22.82发布后，也迁移到了github上，站点在[这里](https://github.com/mkj/dropbear)。除了上面提到的几个版本控制工具，其实还有一些商业工具，比如IBM Rational ClearCase，Microsoft Visual Studio Team Foundaton Server(已改名为Azure DevOps Server)等，不过主要总在一些内部场景，已经逐渐小众了，在这里就不做召开了。在当下git几乎完全占据版本控制工具主流的情况下，我们如何找到新的创新点，是值得思考的问题。