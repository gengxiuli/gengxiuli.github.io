---
layout: post
title:  "Linux Iproute2 Command"
date:   2025-02-27
category: linux
tags:   iproute2
---

最近在使用linux上的iproute2工具的时候，发现很多基础命令在网上得花费很大的力气找到，比如配置ipv6静态路由的方法。而且因为iproute2包含了大量网络功能配置，导致iproute2的配置命令以及格式有很多，命令行自带的Help信息也不容易理解和掌握，所以自己在这里做一些总结和整理，便于自己查找使用。

下面是在shell下输入ip命令回车显示的打印信息，也是iproute2工具ip主命令的帮助提示：
```shell
Usage: ip [ OPTIONS ] OBJECT { COMMAND | help }
       ip [ -force ] -batch filename
where  OBJECT := { address | addrlabel | amt | fou | help | ila | ioam | l2tp |
                   link | macsec | maddress | monitor | mptcp | mroute | mrule |
                   neighbor | neighbour | netconf | netns | nexthop | ntable |
                   ntbl | route | rule | sr | tap | tcpmetrics |
                   token | tunnel | tuntap | vrf | xfrm }
       OPTIONS := { -V[ersion] | -s[tatistics] | -d[etails] | -r[esolve] |
                    -h[uman-readable] | -iec | -j[son] | -p[retty] |
                    -f[amily] { inet | inet6 | mpls | bridge | link } |
                    -4 | -6 | -M | -B | -0 |
                    -l[oops] { maximum-addr-flush-attempts } | -br[ief] |
                    -o[neline] | -t[imestamp] | -ts[hort] | -b[atch] [filename] |
                    -rc[vbuf] [size] | -n[etns] name | -N[umeric] | -a[ll] |
                    -c[olor]}
```

ip [ OPTIONS ] OBJECT { COMMAND | help }格式的含义是ip后面跟的是OPTIONS选项，然后是操作的OBJECT对象，最后是对象支持的COMMAND命令，可以使用help查看对象支持的命令。
注意OPTIONS(带有-格式的字符串)必须在OBJECT的前面，比如ip -s addr会打印正确打印ip地址相关统计信息，但是ip addr -s会就提示Command "-s" is unknown，因为这里-s被解释为了命令字符串。

从概念区分上来说，选项对于所有对象是通用的，比如addr对象可以支持-s统计，neigh对象也可以支持，但是二者的统计信息含义是不一样的，而命令是针对具体对象的，每个对象有自己特定的命令。

我们以配置静态路由为例，说明如何使用iproute2工具，怎样查找命令及配置方法，同时给出一些常用的命令。

使用ip route help就可以看到ip route支持的所有命令格式，注意如果不带help则默认打印ipv4路由表信息。

```shell
Usage: ip route { list | flush } SELECTOR
       ip route save SELECTOR
       ip route restore
       ip route showdump
       ip route get [ ROUTE_GET_FLAGS ] ADDRESS
                            [ from ADDRESS iif STRING ]
                            [ oif STRING ] [ tos TOS ]
                            [ mark NUMBER ] [ vrf NAME ]
                            [ uid NUMBER ] [ ipproto PROTOCOL ]
                            [ sport NUMBER ] [ dport NUMBER ]
       ip route { add | del | change | append | replace } ROUTE
SELECTOR := [ root PREFIX ] [ match PREFIX ] [ exact PREFIX ]
            [ table TABLE_ID ] [ vrf NAME ] [ proto RTPROTO ]
            [ type TYPE ] [ scope SCOPE ]
ROUTE := NODE_SPEC [ INFO_SPEC ]
NODE_SPEC := [ TYPE ] PREFIX [ tos TOS ]
             [ table TABLE_ID ] [ proto RTPROTO ]
             [ scope SCOPE ] [ metric METRIC ]
             [ ttl-propagate { enabled | disabled } ]
INFO_SPEC := { NH | nhid ID } OPTIONS FLAGS [ nexthop NH ]...
NH := [ encap ENCAPTYPE ENCAPHDR ] [ via [ FAMILY ] ADDRESS ]
      [ dev STRING ] [ weight NUMBER ] NHFLAGS
FAMILY := [ inet | inet6 | mpls | bridge | link ]
OPTIONS := FLAGS [ mtu NUMBER ] [ advmss NUMBER ] [ as [ to ] ADDRESS ]
           [ rtt TIME ] [ rttvar TIME ] [ reordering NUMBER ]
           [ window NUMBER ] [ cwnd NUMBER ] [ initcwnd NUMBER ]
           [ ssthresh NUMBER ] [ realms REALM ] [ src ADDRESS ]
           [ rto_min TIME ] [ hoplimit NUMBER ] [ initrwnd NUMBER ]
           [ features FEATURES ] [ quickack BOOL ] [ congctl NAME ]
           [ pref PREF ] [ expires TIME ] [ fastopen_no_cookie BOOL ]
TYPE := { unicast | local | broadcast | multicast | throw |
          unreachable | prohibit | blackhole | nat }
TABLE_ID := [ local | main | default | all | NUMBER ]
SCOPE := [ host | link | global | NUMBER ]
NHFLAGS := [ onlink | pervasive ]
RTPROTO := [ kernel | boot | static | NUMBER ]
PREF := [ low | medium | high ]
TIME := NUMBER[s|ms]
BOOL := [1|0]
FEATURES := ecn
ENCAPTYPE := [ mpls | ip | ip6 | seg6 | seg6local | rpl | ioam6 | xfrm ]
ENCAPHDR := [ MPLSLABEL | SEG6HDR | SEG6LOCAL | IOAM6HDR | XFRMINFO ]
SEG6HDR := [ mode SEGMODE ] segs ADDR1,ADDRi,ADDRn [hmac HMACKEYID] [cleanup]
SEGMODE := [ encap | encap.red | inline | l2encap | l2encap.red ]
SEG6LOCAL := action ACTION [ OPTIONS ] [ count ]
ACTION := { End | End.X | End.T | End.DX2 | End.DX6 | End.DX4 |
            End.DT6 | End.DT4 | End.DT46 | End.B6 | End.B6.Encaps |
            End.BM | End.S | End.AS | End.AM | End.BPF }
OPTIONS := OPTION [ OPTIONS ]
OPTION := { flavors FLAVORS | srh SEG6HDR | nh4 ADDR | nh6 ADDR | iif DEV | oif DEV |
            table TABLEID | vrftable TABLEID | endpoint PROGNAME }
FLAVORS := { FLAVOR[,FLAVOR] }
FLAVOR := { psp | usp | usd | next-csid }
IOAM6HDR := trace prealloc type IOAM6_TRACE_TYPE ns IOAM6_NAMESPACE size IOAM6_TRACE_SIZE
XFRMINFO := if_id IF_ID [ link_dev LINK ]
ROUTE_GET_FLAGS := [ fibmatch ]
```

对于添加路由来说，对应的命令格式为：ip route { **add** | del | change | append | replace } ROUTE,也就是ip route add ROUTE, 
最后的ROUTE的定义格式为：ROUTE := NODE_SPEC [ INFO_SPEC ],其中INFO_SPEC根据具体配置是可选项，而NODE_SPEC的格式为：

```shell
NODE_SPEC := [ TYPE ] PREFIX [ tos TOS ]
             [ table TABLE_ID ] [ proto RTPROTO ]
             [ scope SCOPE ] [ metric METRIC ]
             [ ttl-propagate { enabled | disabled } ]
```

可以看到其中除了PREFIX是必选项之外，其他都是可选项，那么PREFIX的格式是怎样的，这里就查不到了。

这时我们可以在shell下执行man ip route查看ip route更详细的帮助手册，其中除了包含上述帮助信息外，还对于PREFIX有如下描述：

```shell

to TYPE PREFIX (default)
       the destination prefix of the route. If TYPE is omitted, ip assumes type unicast.  Other values of TYPE are listed above.  PREFIX is an IP or IPv6 address
       optionally followed by a slash and the prefix length. If the length of the prefix is missing, ip assumes a full-length host route. There is also a special
       PREFIX default - which is equivalent to IP 0/0 or to IPv6 ::/0.
```

可以看到PREFIX格式是IP或者IPv6地址/前缀长度的形式，如果没有前缀长度，会被认为是全长度的主机路由，另外，default是一个特殊的默认路由简写，等于IPv4中的 0/0 或IPv6中的 ::/0。
看到这里估计你应该可以大致推断出PREFIX的格式了，IPv4或者IPv6的地址格式也就是标准的点分十进制或者IPv6的十六进制表示。

如果看到这里你还有疑问，你还可以继续看到最后，帮助手册还给出了一些配置距离，这里就可以看到真实的用法了。

```shell

EXAMPLES
       ip ro
           Show all route entries in the kernel.

       ip route add default via 192.168.1.1 dev eth0
           Adds a default route (for all addresses) via the local gateway 192.168.1.1 that can be reached on device eth0.

       ip route add 10.1.1.0/30 encap mpls 200/300 via 10.1.1.1 dev eth0
           Adds an ipv4 route with mpls encapsulation attributes attached to it.

       ip -6 route add 2001:db8:1::/64 encap seg6 mode encap segs 2001:db8:42::1,2001:db8:ffff::2 dev eth0
           Adds an IPv6 route with SRv6 encapsulation and two segments attached.

       ip -6 route add 2001:db8:1::/64 encap seg6local action End.DT46 vrftable 100 dev vrf100
           Adds an IPv6 route with SRv6 decapsulation and forward with lookup in VRF table.

       ip -6 route add 2001:db8:1::/64 encap seg6local action End flavors next-csid dev eth0
           Adds an IPv6 route with SRv6 End behavior with next-csid flavor enabled.

       ip -6 route add 2001:db8:1::/64 encap seg6local action End flavors next-csid lblen 48 nflen 16 dev eth0
           Adds an IPv6 route with SRv6 End behavior with next-csid flavor enabled and user-provided Locator-Block and Locator-Node Function lengths.

       ip -6 route add 2001:db8:1::/64 encap ioam6 freq 2/5 mode encap tundst 2001:db8:42::1 trace prealloc type 0x800000 ns 1 size 12 dev eth0
           Adds an IPv6 route with an IOAM Pre-allocated Trace encapsulation (ip6ip6) that only includes the hop limit and the node id, configured for the IOAM namespace 1 and
           a pre-allocated data block of 12 octets (will be injected in 2 packets every 5 packets).

       ip route add 10.1.1.0/30 nhid 10
           Adds an ipv4 route using nexthop object with id 10.
```

至此，你应该可以写出ip route的实际用法了。
对于 iproute2 支持的其他功能，可以按照上述方法找到所需的命令。

2025-03-01更新

在Debian 12上试了下，有以下注意事项
1) man ip address不能写成man ip addr,否则看不到ip address的帮助页信息，而是man ip的帮助信息。这是因为OBJECT中有address和addrlabel都可以匹配man ip addr,但是你直接输入ip addr就是ip address，而不是ip addrlabel。
2) man ip neighbour不能写成man ip neigh、man ip neighbor,否则也是man ip的帮助信息。但是同样的，ip neigh,ip neighbor,ip neighbour都是有效的命令，甚至ip ne, ip n都可以匹配到ip neighbour。
3) 综上所述，在使用ip命令的时候，OBJECT可以简写，甚至只写前几个字母，只要没有歧义，但是man ip 的时候OBJECT必须写全称，否则只打印man ip的帮助信息。
