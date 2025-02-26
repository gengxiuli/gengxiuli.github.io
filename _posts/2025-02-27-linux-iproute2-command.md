---
layout: post
title:  "linux iproute2 command"
date:   2025-02-27
category: linux
tags:   iproute2
---

最近在使用linux上的iproute2工具的时候，发现很多基础命令在网上得花费很大的力气找到，比如配置ipv6静态路由的方法。而且因为iproute2包含了大量网络功能配置，导致iproute2的配置命令以及格式有很多，命令行自带的Help信息也容易理解和掌握，所以自己在这里做一些总结和整理，便于自己已有查找使用。

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
注意OPTIONS(带有-格式的字符串)必须在OBJECT的前面，比如ip addr -s会提示Command "-s" is unknown，但是ip -s addr就打印正确打印ip地址相关统计信息，因为这里-s被解释为了命令字符串。
从概念区分上来说，选项对于所有对象是通用的，比如addr对象可以支持-s统计，neigh对象也可以支持，但是二者的统计信息含义是不一样的，而命令是针对具体对象的，每个对象有自己特定的命令。
