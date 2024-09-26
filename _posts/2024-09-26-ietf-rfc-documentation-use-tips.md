---
layout: post
title:  关于RFC文档的一些研究体会
date:   2024-09-26
category: networking
tags:   ietf rfc
---

1.博客文章[《计算机网络那些标识分配资源》](https://www.cnblogs.com/lionelgeng/p/14520025.html)提到了TCP/IP网络中一些字段的资源分配情况，其实RFC在很早的时候就有过这种统计，而且更加全面，比如[rfc1340](https://tools.ietf.org/html/rfc1340)，[rfc1700](https://tools.ietf.org/html/rfc1700)，但是从[rfc3232](https://www.rfc-editor.org/rfc/rfc3232.txt)开始，这些资源分配被放到在线网站上（[www.iana.org](https://www.iana.org)），且不再提供rfc格式的更新。之所以这样做，很大程度是因为这些资源一直在随着协议的变化在被更新，RFC这种变化慢的标准文档无法及时覆盖这些更新。
但是从另一方面看，从最开始的rfc1340(1992年7月发布)到现在，TCP/IP协议中的字段分配一直是平滑演进的，即新的标准兼容老的标准，老的资源在新的协议下依然可以使用，这对于互联网通信是至关重要的。因此，不论是rfc标准文档还是在线更新，互联网报文格式资源分配的这种设计准则是保持不变的。

2.另外，如果经常阅读或者分析RFC文档，会发现旧的文档会被新的文档更新，虽然新文档会兼容旧文档，但是如果不能及时阅读最新的文档标准，无异于让我们落后于技术的发展和变化。那么如何知道一个rfc被哪些文档更新了呢？一种是可以根据在线rfc最开始的Obsoleted by来判断，但是如果rfc被下载到本地，文件开头是没有这种标记的，道理也很简单，谁也无法预知当时的一个文档会被哪些文档更新。另一种方法是及时查阅rfc-index资源，这是一个在线站点（也可以下载离线版本），里面详细描述了所有的rfc标题，发布时间，状态，替换了哪些rfc以及被哪些rfc更新了，地址为[https://www.rfc-editor.org/rfc-index.html](https://www.rfc-editor.org/rfc-index.html)，[https://www.rfc-editor.org/rfc-index2.html](https://www.rfc-editor.org/rfc-index2.html)是反向排列，最上面是最新的。这样我们就可以及时知道某些协议的最新文档是哪些，既方便又准确，并且可以根据协议的关键字将所有的RFC过滤出来，比如Segment Routing。

3.互联网的快速发展离不开一个个的RFC文档，但是这一个个文档是如何来的呢？是先有了RFC标准，然后各个硬件和软件厂商去实现，还是厂商先根据需求研究实现，然后在实现的过程中讨论验证迭代，最终形成完善的文档，再讲这些文档标准化为RFC？这个问题我想只能去深入分析网络的来龙去脉才能获得，而rfc这一个个文档就是非常好的学习资源。

4.china-pub曾经翻译过一些rfc文档，不过后来没有更新了。目前有一个网站有rfc的一些在线资源，具体在这里[http://rfc.ac.cn/rfcindex.html](http://rfc.ac.cn/rfcindex.html)。其实学习RFC最好的方法就是阅读英文原文，这样既能够阅读最原始的资料，也能学习常用的计算机网络技术用语。rfc在线资源有几个国内的镜像，可以作为备份使用。
 [http://mirrors.zju.edu.cn/rfc/](http://mirrors.zju.edu.cn/rfc/)
 [http://mirrors.nju.edu.cn/rfc/](http://mirrors.nju.edu.cn/rfc/)
 [http://unicom.mirrors.ustc.edu.cn/rfc/](http://unicom.mirrors.ustc.edu.cn/rfc/)

5.为了便于阅读且兼容旧的显示设备，RFC文档的显示格式一直是按照page来管理的，这样可以保证在大多数设备上都可以方便的阅读。不过随着显示设备的发展，这种格式越来越不适合打屏幕阅读了，同时对应的PDF文档没有目录结构，这也是RFC文档阅读起来的一个痛点。好在RFC组织的这些人也认识到了这些问题，因此从RFC8650开始（2019年11月），txt文档不再使用page结构[https://tools.ietf.org/html/rfc8650]，同时对应的pdf文件也有了目录信息，方便阅读了。

2022-05-14更新：
1. 有个将RFC翻译为中文的网站([rfc2cn](http://rfc2cn.com/))，根据该网站说明，由于RFC版权限制，只翻译了RFC2200开始的文章。这个网站的翻译大部分是机器翻译的结果，页面左右分别对应显示英文和中文，包括所有英文字符的编译结果。但这些文档也不是包含所有，SRV6的两篇RFC8754和RFC8986就不再已翻译之列。

2023-01-03更新：
1. 最早看RFC文档一般都是这种形式：https://www.rfc-editor.org/rfc/rfc2460, 也就是文本文件，使用ASCII码表达一些图表等信息。此外还有一些rfc工具可以将使用更加友好的方式查看，比如这个[rfcviewer](https://sourceforge.net/projects/rfcviewer/)。
2. 现在[datatraker](https://datatracker.ietf.org/doc/rfc8986/)看起来符合阅读习惯,因为它还记录了RFC从草案到标准的所有过程，便于了解这个技术的来龙去脉，对于深入学习技术非常有帮助。

2024-09-26更新：
1. ietf上的一个页面[About RFCs](https://www.ietf.org/process/rfcs/)中提到了Publication formats，即发布的文档格式，其中提到了在rfc8650之前，html格式和之后不太一样。直白点说，就是rfc8650之前是将纯文本格式直接渲染成html格式，支持了页面链接点击跳转，而rfc8650之后的html格式，则真正实现了网页效果，支持SVG格式图像，包含相关的过期或者更新rfc,便于手机浏览等。
2. 然后从rfc8650开始，支持了RFCXML格式作为原始编辑格式，而非发布格式，这也意味着RFCXML逐渐在替代Plain Text(纯文本格式)，因为上述html格式需要原始文件的支持，具体细节可以看[这里](https://authors.ietf.org/rfcxml-vocabulary),[这里](https://authors.ietf.org/)和[这里](https://author-tools.ietf.org/)。
