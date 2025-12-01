---
layout: post
title:  "Avici core router"
date:   2025-12-01
category: networking
tags:   avici router
---

在zartbot的微信文章[“网络编程” 还是 “可编程网络”？](https://mp.weixin.qq.com/s?src=11&timestamp=1764545631&ver=6391&signature=YYbOvYHE6kXjY*A9hBCdFZvUxc6hLVpv6T3zKo70jAoTS7ZRGX-*tUCDC5x9V7W*dC2Ejx9bVjY-1UZ0EPylVU95eNp2WGCL1n5uqO4U2kGG*d46wF2ZSSdXatU2wRpG&new=1)中提到过avici路由器：

> VOQ这种技术在日后Nexus7000，ASR9000上继续使用着...当然还有一种技术是华为使用的CIOQ,这个技术来自于Avici，至于Avici的TSR和华为的NE5000是什么关系以及日后的什么什么，自己感兴趣去搜吧。

在2003年的时候，Huawei确实以OEM的方式在中国代理过Avici的核心路由器:

Huawei to Sell Avici Routers in China
<https://convergedigest.com/huawei-to-sell-avici-routers-in-china/?amp=1>

> Avici Systems and Huawei Technologies have entered into an OEM agreement under which Huawei will sell Avici’s routing products in China. Huawei will offer the routers under its own label as part of its networking portfolio.

华为这次又OEM了谁的高端路由器呀，NE80E/NE5000E怎么又那么眼熟呀
<https://www.txrjy.com/forum.php?mod=viewthread&tid=27067&extra=page%3D1&page=2&mobile=yes>

> 美国很多做Core Router的小公司都撑不住了：Caspian倒了；好几个Startup换了CEO；在NASDAQ和Frankfurt同时IPO的Avici正在度日如年；昔日被称为Juniper第二的Procket Networks被Cisco以8500万美金的低廉价格买走了核心技术；名震一时的Tony Li在闲赋半年之后又不甘心地回到Cisco。
>
> 数通领域尤其是核心路由器就是这样，要么自己研发，要么收购，当然这都是烧钱的游戏，只有强者才能支持得下去：Cisco为CRS－1重写了路由平台软件，开发了11块ASIC，包括一个集成了188个RISC的40G NP，累计投资超过5亿美金，历时5年，中间换过几任CTO；Juniper也刚刚发布了支持OC768c的新平台；Alcatel的北美研发中心折腾了很多年没有搞出名堂，最后1.5亿美金收购了TiMetra倒是拣了个现成的大便宜；华为在芯片、软件各个领域奋起直追，目前已经有了很大的起色，虽然CN2招标结果不好。

除了华为，北电网络曾经也代理过Avici的路由器：

电信级核心路由器的扩展性
<http://m.blog.chinaunix.net/uid-20777935-id-543812.html>

> 北电网络在几年前就对核心器进行了战略投资，并拥有小部分Avici公司的资产。2004年1月，北电网络宣布在全球范围内向用户提供对AviciTSR/SSR/QSR等全线核心路由器产品的市场、销售及售后服务等一体化的支持。该系列产品与北电网络MDM网络管理系统平台进行集成，与北电网络的其它数据产品集成到一个网络管理平台，两家公司还在更高的层次上进行产品与技术的合作。TSR/SSR/QSR核心路由器与北电网络的新一代多业务边缘路由器MPE9000产品系列可以为运营商提供一整套新一代运营级IP/MPLS网络解决方案。

> 运营商对IP骨干网络的关键要求在于高可靠性、高可扩展性和线速性能，Avici核心路由器在这些方面具有最领先的技术优势。Avici路由器已经在全球的一些最大型IP网络中（如AT＆T美国国家骨干网）得到了证明。其强劲的软件支持所有主要路由，包括广泛的MPLS及VPN应用，并可以与其他标准进行全面互操作。
　
> Avici核心路由器是在极高的可扩展结构基础上设计而成的，使其能够根据最大型网络的需求来扩展。其核心路由产品包括QSR/SSR/TSR系列，采用完全相同的体系结构、路由控制模块和路由模块。QSR提供10个线路卡插槽，具备200G的容量，能够通过增加机箱扩展到760Gbit/s；SSR提供20个线路卡插槽，具备400G的容量，能够通过增加机箱扩展到1.6Tbit/s以上；TSR提供40个线路卡插槽，具备800G的容量，能够通过增加机箱扩展到11.2Tbit/s以上。不像传统路由器平台，Avici路由器可以扩展到更大的容量，不必硬件升级或成本高昂的群集互联。用户可以根据网络的扩展需要而扩充现有网络的容量，不必废弃原有的交换背板更换一个新的更大的背板，也不必占用线路端口将多个路由器全网状互联。Avici路由器的扩展是通过分布式的交换模块实现的，其交换容量随着核心网络链路的增加而端口随之增加；扩展的系统可以是4个机箱，但只是一个网元系统，由一个路由控制模块进行全系统的监控。

在运营商上Avici路由器也有市场应用：

土耳其电信选择Avici路由器来进行IPMPLS骨干网络建设
> 2005年10月21日，美国马萨诸塞州比尔里卡——Avici系统公司今天宣布土耳其电信公司（Turk Telekom）已经选择了该公司的TSR核心路由器来铺设一条IP/MPLS数据骨干网络，新网络的建成将使这家运营商具备了为土耳其全国的商业和家庭用户提供各种新兴服务的能力。土耳其电信计划在2006年初开通新业务，双方签署的合同分多个阶段，预计到2007年整体网络工程才会完工。

但是，Avici最终还是放弃了核心路由器市场,其中又提到了华为NE5000E和Avici的关系：

分析Avici为何放弃路由器市场
<https://www.51cto.com/article/162529.html>

> 华为的NE5000E核心路由器，当前已经是非常有名了，但对于NE5000和NE5000E的不同和渊源，知道的人就不多了。华为最早进入中国运营商核心路由器市场是靠2004年的广东电信省163骨干网扩容项目，在这个项目中，广东省网出口路由器、顺德、佛山、湛江节点路由器都使用了华为的NE5000路由器，NE5000设备就是OEM Avici的核心路由器产品。2004年中期应该是华为数通产品线最困难时期，高端路由器产品迟迟无法商用，外还有港湾竞争，所以这次NE5000的应用是意义重大。后来在随后的广东电信城域网扩容项目中，华为所销售的NE5000E产品就是自己开发的路由器产品了。这个项目是华为数据产品线在国内上升的开始，也是Cisco高端路由器在运营商市场份额减少的开始。后来的中国电信CN2项目就大规模采用了华为的NE5000E设备。

> 近日了解到Avici在5月份宣布放弃核心路由器市场，将在2007年底停止销售核心路由器产品。然后仔细了解了一下，才发现AT&T是Avici的***一个客户。面对Cisco的高端路由器进入AT&T网络，和其它新技术的挑战，如城域以太网技术，Avici认为核心路由器市场将要发生很大的变化，路由器市场已不足以提供公司所需要的商业增长，尤其是仅有一个客户。

所以总体看，核心路由器的投入非常巨大，除非有稳定的客户收入，否则巨大的投入和有限的收入，会把公司拖垮了。

最近中国移动的高端路由器集采，玩家还是那几家：

中国移动14亿元高端路由器和交换机集采：华为、中兴等四家中标
<https://www.c114.com.cn/news/118/a1299441.html>

其中高端路由器2档，14台，32057万元，平均每台2000万元左右。


