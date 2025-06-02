---
layout: post
title:  Linux kernel net code reading
date:   2025-05-05
category: networking
tags:   linux kernel networking
---

断断续续也阅读了 Linux kernel 中网络子系统的一部分代码，主要集中在目录 net 下。主要的工具有 [https://elixir.bootlin.com/linux](https://elixir.bootlin.com/linux), [https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/) 和 [https://github.com/torvalds/linux](https://github.com/torvalds/linux)。

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
ip_rcv通过调用ip_rcv_core完成ip头有效性校验，再调用ip_rcv_finish进入收包处理流程。在ip_rcv_finish中，通过ip_rcv_finish_core完成路由查找，具体说是通过ip_route_input_noref决定是上送本机还是进行转发，分别设置dst.input为[ip_local_deliver](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_input.c#L240)或者[ip_forward](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_forward.c#L86)。最后，在ip_rcv中通过dst_input调用进入上面dst.input的事件处理流程。

**文件**：
[net/ipv4/ip_output.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_output.c)

*主要函数*：

[ip_output](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_output.c#L423)
在rt_dst_alloc中会将dst.output初始化为ip_output, 后面调用dst_output的地方，实际就是进入ip_output中进行处理。而ip_output最终会调用[ip_finish_output](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_output.c#L311)。ip_finish_output的调用路径图如下：

ip_finish_output

-->__ip_finish_output

---->dst_output NAT策略处理之后，需要重新再走一遍output流程。

---->ip_finish_output_gso gso处理流程

------>ip_finish_output2

------>ip_fragment

---->ip_fragment 需要分片处理的报文

------>ip_finish_output2

---->ip_finish_output2


可见最后都会进入[ip_finish_output2](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_output.c#L187)，该函数的调用路径如下：

ip_finish_output2

-->ip_neigh_for_gw [route.h](https://elixir.bootlin.com/linux/v5.10.70/source/include/net/route.h#L379)

---->ip_neigh_gw4

---->ip_neigh_gw6

-->neigh_output [neighbour.h](https://elixir.bootlin.com/linux/v5.10.70/source/include/net/neighbour.h#L502)

---->neigh_hh_output

---->n->output

[ip_queue_xmit](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_output.c#L545)
tcp层发包最终调用的接口，该接口内部再调用__ip_queue_xmit，后者内部先通过ip_route_output_ports查找路由，最后通过ip_local_out发包。ip_local_out内部最终调用dst_output发包。

**文件**:
[net/ipv4/tcp_ipv4.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_ipv4.c)

*主要函数*:

[tcp_v4_rcv](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_ipv4.c#L1916)
在[af_inet.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/af_inet.c#L1732)中注册的tcp处理函数,根据搜索结果，在[ip_protocol_deliver_rcu](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_input.c#L187)中被调用，其实是通过handler指针调用的，同时通过INDIRECT_CALL_2实现间接调用。tcp_v4_rcv做完报文有效性检查后，会调用__inet_lookup_skb查找是否有src+dst对应的socket，最后会同步或异步(tcp_add_backlog)调用tcp_v4_do_rcv。

[tcp_v4_do_rcv](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_ipv4.c#L1655)
tcp_v4_do_rcv会根据tcp的不同状态进行处理，最终会调用[tcp_rcv_state_process](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_input.c#L6312)处理报文。

[tcp_v4_early_demux](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_ipv4.c#L1717)
在ip_rcv_finish_core中做路由查找之前，当配置了sysctl_ip_early_demux支持通过tcp_v4_early_demux做tcp的早期解复用处理，及tcp的快速处理。

**文件**:
[net/ipv4/tcp_input.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_input.c)

*主要函数*:

[tcp_rcv_state_process](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_input.c#L6312)
根据TCP的不同状态处理TCP报文，当非established状态的时候触发tcp状态机，否则使用tcp_data_queue将报文放入队列中，后续由用户态调用的recv/read调用处理。

**文件**:
[net/ipv4/tcp_output.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_output.c)

*主要函数*:

[tcp_write_xmit](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_output.c#L2595)
发送TCP报文的接口，主要提供给__tcp_push_pending_frames和tcp_push_one使用，后两者会在tcp.c中被调用。

[tcp_transmit_skb](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_output.c#L1419)
TCP内部及状态机报文发送接口，主要在tcp_output.c内部使用，tcp_write_xmit也是通过这个接口发包的。最终会调用queue_xmit回调完成报文的发送，具体的接口ipv4和ipv6分别对应ip_queue_xmit和inet6_csk_xmit。

**文件**:
[net/ipv4/tcp.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp.c)

*主要函数*:

[tcp_sendmsg](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp.c#L1442)
用户态send/write/sendmsg/sendto最终调用的发送接口，最终调用tcp_sendmsg_locked。

[tcp_sendmsg_locked(](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp.c#L1189)
实现tcp报文发送，支持零拷贝方式，支持根据MSS分片处理，通过tcp_write_queue_tail读取sk_write_queue,即tcp的发送队列，当push的时候通过[__tcp_push_pending_frames](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_output.c#L2856)或者[tcp_push_one](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp_output.c#L2874)完成发包。通过skb_entail将skb放入sk_write_queue，以便实现异步发包。

[tcp_recvmsg](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/tcp.c#L2019)
用户态recv/read/recvmsg/recvfrom最终调用的接收接口，通过skb_peek_tail从sk_receive_queue获得数据，即tcp的接收队列。

**文件**:
[net/ipv4/arp.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/arp.c)

*主要函数*:

[arp_send](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/arp.c#L322)
向指定目的IP发送arp，type决定的发送报文的类型，支持请求和响应报文。具体通过arp_create创建skb,通过arp_xmit发包。有些模块在发包之前有自己的处理流程，比如bound模块，参考[这里(]https://elixir.bootlin.com/linux/v5.10.70/source/drivers/net/bonding/bond_main.c#L2704)，或者vxlan模块，参考[这里](https://elixir.bootlin.com/linux/v5.10.70/source/drivers/net/bonding/bond_main.c#L2704)。所以将组包和发包进行了拆分。

[arp_create](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/arp.c#L523)
创建arp报文，根据输入的参数填充skb中的相关字段结构。

[arp_xmit](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/arp.c#L638)
通过调用arp_xmit_finish发包，而arp_xmit_finish中直接调用了dev_queue_xmit进行发包。

下面是arp的一些处理函数，对于理解arp流程会有重要帮助。

[arp_constructor](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/arp.c#L222)
构建neighbour结构体，也就是arp表项的构建，由neigh邻居子系统在创建neigh的时候调用。

[arp_solicit](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/arp.c#L332)
发送arp请求，内部最终通过arp_send_dst发送。arp_send_dst内部通过arp_create和arp_xmit完成发包。由neigh邻居子系统在发送请求的时候调用。

[arp_process](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/arp.c#L677)
处理接收到的arp报文，包括邻居发送的请求报文以及响应报文。

[arp_rcv](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/arp.c#L942)
向协议栈注册的arp收包处理入口，完成arp报文格式基本检查已经netfilter处理后，交给arp_process处理。

**文件**:
[net/core/neighbour.c](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c)

*主要函数*:

[__neigh_create](https://elixir.bootlin.com/linux/v5.10.70/C/ident/__neigh_create)
创建neighbour表项。ipv4和ipv6通过该接口创建neigh表项，请参考[这里](https://elixir.bootlin.com/linux/v5.10.70/source/include/net/route.h#L374)和[这里](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv6/ip6_output.c#L142)。在ipv4的流程中，会调用arp中的arp_constructor构建neighbour结构，

[neigh_destroy](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L835)
删除neighbour表项。

[neigh_update](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L1419)
更新neighbour状态，由arp或者ipv6 nd根据接收到的报文调用。

[neigh_xmit](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L2994)
查找对应地址的neigh表项，如果不存在则创建，然后通过neigh子系统中的output回调进行探测或者发包。可以看出，这是没有 cache 支持的neigh发送流程，什么模块会使用呢: mpls。mpls没有类似于ip转发的hh缓存,所以在[mpls_xmit](https://elixir.bootlin.com/linux/v5.10.70/source/net/mpls/mpls_iptunnel.c#L36)和[mpls_forward](https://elixir.bootlin.com/linux/v5.10.70/source/net/mpls/af_mpls.c#L341)中都是通过neigh_xmit发送报文的。

[neigh_suspect](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L873)
neigh状态可疑的时候，关闭快速发送通道。

[neigh_connect](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L885)
neigh状态正常的时候，打开快速发送通道。

[neigh_resolve_output](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L1477)
解析neigh状态后发送，通过neigh_event_send->__neigh_event_send触发定时器或者立即发送arp请求，如果dev的header_ops中cache有效，这里对于eth设备对应的是[eth_header_cache](https://elixir.bootlin.com/linux/v5.10.70/source/net/ethernet/eth.c#L233)，同时hh.hh_len为0表示没有缓存，则调用 neigh_hh_init 更新hh缓存，然后根据neigh填充二层头。最后通过dev_queue_xmit发包。

[neigh_hh_init](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L1458)
调用 neigh 对应的 dev 中的 header_ops->cache更新 hh 缓存，cache 对于 eth 对应的是 [eth_header_cache](https://elixir.bootlin.com/linux/v5.10.70/source/net/ethernet/eth.c#L233), 在 [ether_setup](https://elixir.bootlin.com/linux/v5.10.70/source/net/ethernet/eth.c#L361)中通过[eth_header_ops](https://elixir.bootlin.com/linux/v5.10.70/source/net/ethernet/eth.c#L347)注册的。

[neigh_connected_output](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L1512)
当前已完成neigh探测后，当没有支持hh cache的时候进行发包。对于eth设备来说，因为支持hh，所有不走这里。这是一种通用接口，一些没有支持邻居缓存的设备使用这种方法完成状态解析后的发包。从 neigh_connected_output 和 neigh_resolve_output 的代码可以看出，前者相对于后者少了缓存的流程。

[neigh_direct_output](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L1535)
直接调用dev_queue_xmit进行发包。当dev设备结构中的header_ops没有赋值的时候使用这种发送模式，比如一些虚拟设备。对于eth设备，在[ether_setup](https://elixir.bootlin.com/linux/v5.10.70/source/net/ethernet/eth.c#L361)中进行了赋值。

[neigh_parms_alloc](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L1683)
neigh参数初始化，同时绑定dev设备和neigh表(arp,nd等)。

[neigh_table_init](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L1683)
neigh邻居表初始化，arp和ipv6 nd调用初始化arp表和ip nd表。

[neigh_add](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L1862)
[neigh_delete](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L1797)
[neigh_get](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L2876)
[neightbl_dump_info](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L2374)
[neightbl_set](https://elixir.bootlin.com/linux/v5.10.70/source/net/core/neighbour.c#L2188)
上面5个函数是向rtnetlink注册的回调函数，分别实现neigh的添加，删除，获取, 导出和配置操作。

**文件**:
[net/core/neighbour.h](https://elixir.bootlin.com/linux/v5.10.70/source/include/net/neighbour.h)

*主要函数*:

[neigh_output](https://elixir.bootlin.com/linux/v5.10.70/source/include/net/neighbour.h#L502)
邻居子系统提供的发送接口，以 inline 的形式定义在 neighbour.h 中。

```c
static inline int neigh_output(struct neighbour *n, struct sk_buff *skb,
			       bool skip_cache)
{
	const struct hh_cache *hh = &n->hh;

	if ((n->nud_state & NUD_CONNECTED) && hh->hh_len && !skip_cache)
		return neigh_hh_output(hh, skb);
	else
		return n->output(n, skb);
}
```

当 neigh 的状态为 NUD_CONNECTED，且 h h缓存长度非0， 且未设置 skip_cache，则使用 neigh_hh_output 进行缓存发送，否则使用neigh注册的output回调，其实就是neigh_resolve_output。 neigh_output被[ipv4](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_output.c#L230)和[ipv6](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv6/ip6_output.c#L145)调用, 注意这没有mpls, 因此目前mpls没有neigh缓存机制发包。其实bridge在[br_nf_pre_routing_finish_bridge](https://elixir.bootlin.com/linux/v5.10.70/source/net/bridge/br_netfilter_hooks.c#L281)中还通过neigh子系统提供的[neigh_hh_bridge](https://elixir.bootlin.com/linux/v5.10.70/source/include/net/neighbour.h#L449)接口实现了通过缓存机制发包。

[neigh_hh_output](https://elixir.bootlin.com/linux/v5.10.70/source/include/net/neighbour.h#L462)
neigh 子系统提供的缓存发送接口，复制 hh 缓存中的数据到 skb 中，然后使用 dev_queue_xmit 发包。neigh_hh_output目前只被neigh_output调用。
缓存数据是何时初始化的呢？neigh_resolve_output中。