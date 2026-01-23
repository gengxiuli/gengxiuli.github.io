---
layout: post
title:  "Service OUI information"
date:   2026-01-23
category: networking
tags:   mac oui
---

网络设备，几乎都有一个唯一的，且独一无二的MAC地址，比如我们平时使用的手机，电脑。在手机上，蓝牙设备和WLAN设备有各自独立的MAC地址，具体可以参考iOS或者Android的系统信息页。

MAC地址是一个48bit，可以表示为6个字节的序列。主要分为两个部分，厂商ID和设备ID。厂商ID以前叫做OUI (Organizationally Unique Identifier) ，在IEEE的网站上最新的叫法是[MAC Address Block Large (MA-L)](https://standards.ieee.org/products-programs/regauth/oui/)。之所以这么修改，是因为3个字节的厂商ID有时候太大了，因为它可以支持2^24，大约是1600万个设备，很多小厂商根本用不了这么多，多花费钱不说，还造成了很大的资源浪费。所以又推出了[MAC Address Block Medium (MA-M)](https://standards.ieee.org/products-programs/regauth/oui28/)和[MAC Address Block Small (MA-S)](https://standards.ieee.org/products-programs/regauth/oui36/),厂商ID长度分别是28和36bit，这样对应的设备数量就是2^20(大约100万个)和2^12(大约4000个)。以前我们通过MAC地址的前3个字节就可以判断厂家，后续如果使用的是MA-M或者MA-S格式，则需要更多的尾数了。

最近在使用各种云服务做网络配置的时候，发现虽然有网卡接口，但是因为在虚拟环境中，网卡的MAC地址并不是通常意义上的MAC地址，即OUI部分可能并不代表厂商信息，或者根本找不到信息。

比如某云厂商使用的MAC地址如下,使用ip addr命令查询：

2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:16:3e:xx:xx:xx brd ff:ff:ff:ff:ff:ff

根据Wireshark提供的在线查询工具：<https://www.wireshark.org/tools/oui-lookup.html>
我们查找的OUI信息是：00:16:3E Xensource, Inc.
Xen是一种虚拟化技术，Xensource的介绍可以看这里：<https://wiki.xenproject.org/wiki/XenSource>
通过这个信息，我们打开可以猜到底层用的是Xen虚拟化技术，除了Xen，目前主流的虚拟化技术还有Hyper-V,KVM,具体看下面这篇文章:
[Xen和KVM等四大虚拟化架构对比分析](https://support.huawei.com/enterprise/zh/knowledge/EKB1002005920)

另一家云厂商使用的MAC地址如下：

2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 52:54:00:xx:xx:xx brd ff:ff:ff:ff:ff:ff

我们查询52:54:00:xx:xx:xx，发现找不到结果。我们再到IEEE的官网查一下，地址是：
<https://regauth.standards.ieee.org/standards-ra-web/pub/view.html#registries>
发现还是找不到结果，注意在IEEE网站查询的时候，要使用52:54:00或者52-54-00查询，而且类型选择Assignment。
这说明云主机使用的是一个非注册MAC地址，或者是具有隐私保护的OUI信息。

我们再看一下网关的MAC地址，使用ip neigh命令查询，分别得到上面两家云主机的网关地址和对应的MAC地址：

x.x.x.x dev eth0 lladdr ee:ff:ff:xx:xx:xx REACHABLE
x.x.x.x dev eth0 lladdr fe:ee:8f:xx:xx:xx REACHABLE

再次分别查询ee:ff:ff和fe:ee:8f，发现都无法查到信息。虽然我们有理由推测这些OUI使用了隐私保护，但是ee:ff:ff这个地址怎么看都像一个非分配地址，而更像一个保留地址，比如ff:ff:ff。

因为MAC地址和IP地址一样，都涉及一些隐私，所以直接暴露MAC地址或者OUI会造成一些技术泄露或者安全风险，上面云厂商的OUI信息处理也正是基于这些考虑。另外，在虚拟化环境中，如果所有网络设备都可以受控，其实也可以不必使用公开的OUI信息为设备做标识。

根据这篇文章<https://superuser.com/questions/907827/private-mac-address>,作者给出了私有MAC地址的格式：

>Private MAC addresses are often found in embedded systems that do not have an official address. Many cheap "credit card computers" such as the Raspberry Pi must generate their own address to operate without an official, manufacturer-assigned address.
>
>For you interest: Private MAC addresses can be identified by having the second-least-significant bit of the most significant byte set. (And as unicast addresses, they must not have the least significant bit set.) That means any addres matching any pattern below is private.
>
>x2:xx:xx:xx:xx:xx
>x6:xx:xx:xx:xx:xx
>xA:xx:xx:xx:xx:xx
>xE:xx:xx:xx:xx:xx

所以上述ee:ff:ff和fe:ee:8f都符合私有MAC地址特征。

其实在[RFC9542](https://datatracker.ietf.org/doc/html/rfc9542#name-48-bit-mac-identifiers-ouis)中也对OUI、MAC块做了解释和说明：其中首个字节
格式如下：

```shell

  0  1  2  3  4  5  6  7  
+--+--+--+--+--+--+--+--+
| .  .  .  .  Z  Y  X  M
```
> There are bits within the initial octet of an IEEE MAC address that have special significance [IEEE802_OandA], as described as follows:
> 
> M bit -
>This bit is frequently referred to as the "group" or "multicast" bit. If it is zero, the MAC address is unicast. If it is a one, the address is groupcast (multicast or broadcast). This meaning is independent of the values of the X, Y, and Z bits.
>X bit -
>This bit is also called the "universal/local" bit (formerly called the Local/Global bit). If it is zero, the MAC address is a global address under the control of the owner of the IEEE-assigned prefix. Previously, if it was a one, the MAC address was considered "local" and under the assignment and control of the local network operator (but see Section 2.3). If it is a one and if the IEEE 802 Structured Local Address Plan (SLAP) is in effect, the nature of the MAC address is optionally determined by the Y and Z bits, as described below.

IANA的相关信息页面如下：[IANA OUI Ethernet Numbers](https://www.iana.org/assignments/ethernet-numbers/ethernet-numbers.xhtml#ethernet-numbers-2)