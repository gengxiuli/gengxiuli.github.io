---
layout: post
title:  "Linux kernel neigh confirm"
date:   2026-02-26
category: networking
tags:   neigh
---

在内核函数[ip_finish_output2](https://elixir.bootlin.com/linux/v5.10.70/source/net/ipv4/ip_output.c#L224)中，在通过ip_neigh_for_gw获取到neigh信息之后（这里可能是首次创建，也可能是使用已有的neigh），在使用neigh_output发送报文之前，会调用sock_confirm_neigh对neigh的confirmed时间和sock的sk_dst_pending_confirm标志进行更新，在更新之前会判断skb是否设置了dst_pending_confirm。

在内核2017年的这个[replace-dst_confirm](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/commit/?id=29ba6e7400a317725bdfb86a725d1824447dbcd7)分支中,在skbuff结构中引入了dst_pending_confirm标志，并增加了set/get接口skb_set_dst_pending_confirm/skb_get_dst_pending_confirm。同时将调用dst_confirm的地方替换为了dst_confirm_neigh，并将dst_confirm的函数实现修改为空。dst_confirm_neigh会调用dst->ops->confirm_neigh，而这又分别对应ipv4/ipv6的ipv4_confirm_neigh/ip6_confirm_neigh。在sock结构中引入了sk_dst_pending_confirm标志，并增加了设置接口sk_dst_confirm，sock_confirm_neigh，前者将sk_dst_pending_confirm从0修改为1，后者在skbuff中dst_pending_confirm为1的时候，将sk_dst_pending_confirm 修改为0.

在上述patch修改之前，协议会基于dst_entry更新confirm信息，但是当dst_entry对应多个nexthop的时候，一个nexthop的收到的报文会导致其他nexthop的confirm也会更新。

在Ubuntu的Bug库中也可以看到这个修改[Neighbour confirmation broken, breaks ARP cache aging](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1715812)。文中详细描述了这个Bug的现象和修复结果。这个问题的修复也是合入了上一段的patch[replace-dst_confirm](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/commit/?id=29ba6e7400a317725bdfb86a725d1824447dbcd7)。这个bug说明的是，当一个host的MAC地址变化的时候，如果这个host和其他host共享了route,也就是dst_entry，当从其他host收到报文进行confirm的时候也confirm这个host，导致这个host还使用了之前的MAC地址。

所以上述patch修改的目的是，将neigh confirm的实现放到了sock和skbuff中，而不是dst_entry，因为dst_entry和skbuff可能是一对多的关系。

在[[PATCHv4 net-next 0/7] net: dst_confirm replacement](https://www.spinics.net/lists/linux-rdma/msg45907.html)这个页面，可以看到每个patch对应的标题，这对于检索每个改动很有帮助。

```
 Julian Anastasov (7):
  sock: add sk_dst_pending_confirm flag
  net: add dst_pending_confirm flag to skbuff
  sctp: add dst_pending_confirm flag
  tcp: replace dst_confirm with sk_dst_confirm
  net: add confirm_neigh method to dst_ops
  net: use dst_confirm_neigh for UDP, RAW, ICMP, L2TP
  net: pending_confirm is not used anymore
```

上述patch还提到了MSG_CONFIRM标志，MSG_CONFIRM在<https://www.man7.org/linux/man-pages/man2/send.2.html>中介绍，具体如下：
>        MSG_CONFIRM (since Linux 2.3.15)
              Tell the link layer that forward progress happened: you got
              a successful reply from the other side.  If the link layer
              doesn't get this it will regularly reprobe the neighbor
              (e.g., via a unicast ARP).  Valid only on SOCK_DGRAM and
              SOCK_RAW sockets and currently implemented only for IPv4
              and IPv6.  See arp(7) for details.

也就是说，设置这个标志位，可以触发SOCK_DGRAM和SOCK_RAW的arp探测过程。这个标志在代码实现上，其实是调用skb_set_dst_pending_confirm设置了skb的dst_pending_confirm标志。