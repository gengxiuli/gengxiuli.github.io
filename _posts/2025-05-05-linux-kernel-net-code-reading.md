---
layout: post
title:  Linux kernel net code reading
date:   2025-05-05
category: networking
tags:   linux kernel networking
---

断断续续也阅读了 Linux kernel 中网络子系统的一部分代码，主要集中在目录 net 下。主要的工具有 https://elixir.bootlin.com/linux, https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/ 和 https://github.com/torvalds/linux。具体这几个工具的比较可以参考[这里](https://thinkdancer.net/2025/04/06/linux-kernel-code-review-online/)。下面是目前主要阅读的一些源文件，并根据自己的阅读经验总结了主要功能，后面可以集中时间专门总结一下网络子系统。

**文件**：
[net/core/dev.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/dev.c)

*主要函数*：

[netif_rx](https://elixir.bootlin.com/linux/v5.10.70/C/ident/netif_rx)
驱动层收包之后，申请skb后交给网络协议栈的入口。支持在硬件中断中处理报文，实际是通过发送给queue调度处理。

[netif_receive_skb](https://elixir.bootlin.com/linux/v5.10.70/C/ident/netif_receive_skb)
驱动层收包之后，申请skb后交给网络协议栈的入口。和netif_rx的区别是，支持在软中断中处理报文，即直接在当前上下文中处理，一般用在NAPI的poll函数中。

[napi_gro_receive](https://elixir.bootlin.com/linux/v5.10.70/C/ident/napi_gro_receive)
驱动层收包之后，通过napi方式触发poll轮询收包，之后使用该接口发送给网络协议栈。

[__napi_schedule](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/dev.c#L6403)
在驱动中断处理中，通过该函数触发NAPI的调度，也就是调用驱动的poll轮询回调。

[netif_napi_add](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/dev.c#L6722)
驱动程序通过该接口向napi注册poll回调函数。

[napi_complete_done](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/dev.c#L6465)
驱动在poll中完成收包后，通过该接口告诉NAPI完成收包。有部分驱动调用的是napi_complete,其实是封装的napi_complete_done

[dev_queue_xmit](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/dev.c#L4185)
驱动层发包函数，这是驱动层对上层提供的统一发包入口，比如neighbour邻居子系统通过调用该函数实现发包。这个函数通过dev_hard_start_xmit->xmit_one->netdev_start_xmit->__netdev_start_xmit调用路径，最终调用具体驱动程序注册的ndo_start_xmit回调完成发包。

**文件**：
[net/ipv4/devinet.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/devinet.c)

*主要函数*：

[inet_rtm_newaddr](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/devinet.c#L928)
通过netlink添加IP地址的处理函数。

[inet_rtm_deladdr](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/devinet.c#L644)
通过netlink删除IP地址的处理函数。

[devinet_ioctl](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/devinet.c#L1009)
通过ioctl方式添加或者删除IP地址的处理函数。

**文件**：
[ip_input.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_input.c)

*主要函数*：

[ip_rcv](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_input.c#L530)
网络协议栈IP层收包入口，具体是在[af_inet.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/af_inet.c#L1934)中注册，在[dev.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/dev.c#L5344)中被调用。注意在被调用的实现中，使用了 INDIRECT_CALL_INET，而且在参数中可以看到可能被回调的函数ip_rcv或ipv6_rcv,这是一种间接调用方式，可以防止一些潜在攻击。
ip_rcv通过调用ip_rcv_core完成ip头有效性校验，再调用ip_rcv_finish进入收包处理流程。在ip_rcv_finish中，通过ip_rcv_finish_core完成路由查找，具体说是通过ip_route_input_noref决定是上送本机还是进行转发，分别设置dst.input为ip_local_deliver或者ip_forward。最后，在ip_rcv中通过dst_input调用进入上面dst.input的时间处理流程。

**文件**
[ip_output.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_output.c)

*主要函数*

[ip_output](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_output.c#L423)
在rt_dst_alloc中会将dst.output初始化为ip_output,后面调用dst_output的地方，实际就是进入ip_output中进行处理。而ip_output最终会调用ip_finish_output。ip_finish_output的调用路径图如下：

ip_finish_output

-->__ip_finish_output

---->dst_output NAT策略处理之后，需要重新再走一遍output流程。

---->ip_finish_output_gso gso处理流程

------>ip_finish_output2

------>ip_fragment

---->ip_fragment 需要分片处理的报文

------>ip_finish_output2

---->ip_finish_output2


可见最后都会进入ip_finish_output2，该函数的调用路径如下：

ip_finish_output2

-->ip_neigh_for_gw [route.h](https://elixir.bootlin.com/linux/v5.10.70/source/include/net/route.h#L379)

---->ip_neigh_gw4

---->ip_neigh_gw6

-->neigh_output [neighbour.h](https://elixir.bootlin.com/linux/v5.10.70/source/include/net/neighbour.h#L502)

---->neigh_hh_output

---->n->output

