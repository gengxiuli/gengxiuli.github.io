---
layout: post
title:  Linux kernel net code reading
date:   2025-05-05
category: networking
tags:   linux kernel networking
---

断断续续也阅读了 Linux kernel 中网络子系统的一部分代码，主要集中在目录 net 下。主要的工具有 https://elixir.bootlin.com/linux, https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/ 和 https://github.com/torvalds/linux。

具体这几个工具的比较可以参考[这里](https://thinkdancer.net/2025/04/06/linux-kernel-code-review-online/)。下面是目前主要阅读的一些源文件，并根据自己的阅读经验总结了主要功能，后面可以集中时间专门总结一下网络子系统。

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
[net/ipv4/ip_input.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_input.c)

*主要函数*：

[ip_rcv](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_input.c#L530)
网络协议栈IP层收包入口，具体是在[af_inet.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/af_inet.c#L1934)中注册，在[dev.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/dev.c#L5344)中被调用。注意在被调用的实现中，使用了 INDIRECT_CALL_INET，而且在参数中可以看到可能被回调的函数ip_rcv或ipv6_rcv,这是一种间接调用方式，可以防止一些潜在攻击。
ip_rcv通过调用ip_rcv_core完成ip头有效性校验，再调用ip_rcv_finish进入收包处理流程。在ip_rcv_finish中，通过ip_rcv_finish_core完成路由查找，具体说是通过ip_route_input_noref决定是上送本机还是进行转发，分别设置dst.input为[ip_local_deliver](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_input.c#L240)或者[ip_forward](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_forward.c#L86)。最后，在ip_rcv中通过dst_input调用进入上面dst.input的时间处理流程。

**文件**：
[net/ipv4/ip_output.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_output.c)

*主要函数*：

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

**文件**:
[tcp_ipv4.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_ipv4.c)

*主要函数*:

[tcp_v4_rcv](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_ipv4.c#L1916)
在[af_inet.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/af_inet.c#L1732)中注册的tcp处理函数,根据搜索结果，在[ip_protocol_deliver_rcu](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_input.c#L187)中被调用，其实是通过handler指针调用的，同时通过INDIRECT_CALL_2实现间接调用。tcp_v4_rcv做完报文有效性检查后，会调用__inet_lookup_skb查找是否有src+dst对应的socket，最后会同步或异步(tcp_add_backlog)调用tcp_v4_do_rcv。

[tcp_v4_do_rcv](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_ipv4.c#L1655)
tcp_v4_do_rcv会根据tcp的不同状态进行处理，最终会调用[tcp_rcv_state_process](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_input.c#L6312)处理报文。

[tcp_v4_early_demux](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_ipv4.c#L1717)
在ip_rcv_finish_core中做路由查找之前，当配置了sysctl_ip_early_demux支持通过tcp_v4_early_demux做tcp的早期解复用处理，及tcp的快速处理。

**文件**:
[tcp_input.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_input.c)

*主要函数*:

[tcp_rcv_state_process](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_input.c#L6312)
根据TCP的不同状态处理TCP报文，当非established状态的时候触发tcp状态机，否则使用tcp_data_queue将报文放入队列中，后续由用户态调用的recv/read调用处理。

**文件**:
[tcp_output.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_output.c)

*主要函数*:

[tcp_write_xmit](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_output.c#L2595)
发送TCP报文的接口，主要提供给__tcp_push_pending_frames和tcp_push_one使用，后两者会在tcp.c中被调用。

[tcp_transmit_skb](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_output.c#L1419)
TCP内部及状态机报文发送接口，主要在tcp_output.c内部使用，tcp_write_xmit也是通过这个接口发包的。

**文件**:
[tcp.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp.c)

*主要函数*:
[tcp_sendmsg](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp.c#L1442)
用户态send/write/sendmsg/sendto最终调用的发送接口，最终调用tcp_sendmsg_locked。

[tcp_sendmsg_locked(](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp.c#L1189)
实现tcp报文发送，支持零拷贝方式，支持根据MSS分片处理，最终通过[__tcp_push_pending_frames](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_output.c#L2856)或者[tcp_push_one](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_output.c#L2874)完成发包。