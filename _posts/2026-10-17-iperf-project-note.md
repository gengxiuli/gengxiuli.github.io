---
layout: post
title:  "Iperf project note"
date:   2026-10-17
category: networking
tags:   iperf
---

## 项目站点
 
 <https://iperf.fr/>

## 代码库

<https://github.com/esnet/iperf>

## 使用文档

<https://iperf.fr/iperf-doc.php>

## 公共iperf服务器

<https://iperf.fr/iperf-servers.php>

## 一些注意

1. iperf代码原生只支持linux,BSD等系统，对于windows的支持是通过cygwin实现的，所以有些cgywin没有支持的特性，在windows下也是不支持的。
2. server同一时间只能接受一个client的链接，测试完成才能接收下一次client连接。这种限制其实不是功能支持不了，可能是带宽限制，因为无法在server侧保证对多个client都有足够的带宽可用，但是iperf2支持这种功能。
3. 现在的iperf默认都是指iperf3，它和iperf2存在很多差异，不能兼容，也就是不能client和server一个用iperf3一个用iperf2。
4. 几乎所有的参数都在client端设置，server端只需要执行iperf -s就可以启动，client和server之间自己交互配置的参数。
5. iperf的代码可以很容易的集成到其他源码中，为此源码中还特地展示了一个示例(<https://github.com/esnet/iperf/tree/master/examples>),其中包含客户端文件mic.c和服务器端文件mis.c。
6. iperf测试的流量默认是单向从client发给server，当设置参数-R的时候变成server发给client，当设置--bidir的时候，client和server之间互相发送流量。
7. iperf默认只有1条流量，配置双向则共2条流量，可以在客户端通过-P参数设置并行多条流量。
8. client和server之间通过json文件格式交换配置参数和计算结果，也就是发送端把上述信息序列化到json文件中，然后实际发送的是字符串，接收端再从字符串恢复成json，然后从上述json文件中反序列化数据。
9. 当使用-b参数配置需要测量的带宽时，iperf使用了比较复杂的方法计算应该发送的报文数量，具体可以参考相关代码。
10. client和server之间的状态机主要由server端驱动，client根据当前的状态做具体的处理，只有当测试完成后client会给server发送测试完成标志，驱动server端的状态结束。
11. iperf支持tcp, udp和sctp这三种传输层协议，每种协议需要支持listen,accept,connect,send，recv和init这几种功能。iperf框架本身是用tcp做控制通道报文传输，其框架实现中对传输协议没有具体依赖，都是通过这几个回调函数来触发的，类似于把传输协议做成了组件的形式。
12. iperf中的定时器timer是自己实现的，timer定时器处理和超时计算是在主线程中处理中触发的。