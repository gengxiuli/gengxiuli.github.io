---
layout: post
title:  Building FRR
date:   2024-09-25
category: networking 
tags:   frr 
---

FRR官网的这篇文章[Building FRR Ubuntu 22.04 LTS](https://docs.frrouting.org/projects/dev-guide/en/latest/building-frr-for-ubuntu2204.html)，几乎可以完全step by step地参考执行，就可以在Ubuntu 22.04上编译出frr了。需要关注的几点是:
1. libyang实测可以使用源码编译方式安装。
2. 根据需要安装mpls内核模块，打开转发支持。
3. 最后一步选择哪些协议可以全部修改为yes。
4. frr命令行的入口是执行sudo vtysh。
5. 保证编译过程网络畅通，因为需要在线安装大量的依赖模块。

从源码编译开始需要安装很多依赖程序，但是好处是可以跟踪主线版本的最新改动，也可以根据需要自行定制修改，还可以跟踪或添加打印信息来完成代码实现的学习，可谓是先苦后甜。

当然，如果只想简单使用一下frr的功能或者作为陪测设备，那么可以直接执行sudo apt install frr,然后选择y,来安装发行版配套的frr程序。

需要注意的是，不论上面哪一种安装方法，最后都要根据需要启动相关进程，因为frr中的不同协议功能在不同进程中实现，做到了很好的隔离，所以在使用中也要自己启动对应的协议进程。