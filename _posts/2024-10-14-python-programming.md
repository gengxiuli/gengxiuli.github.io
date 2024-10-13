---
layout: post
title:  "Python 编程"
date:   2024-10-14
category: programming
tags:   python
---

工作这些年的主力开发语言是 C/C++，确切地说是以 C 语言为主。最近一些年由于在 Linux 下开发，Shell 脚本语言用的也多了一些，各种编译构建，服务加载，系统监控等都可以通过 Shell 来实现。除此之外，在设备脚本执行，自动化测试等场景也涉及到了 Python，但是相对于 C 和 Shell，使用量还是少很多。

最近在深入研究交换机开源操作系统，发现其中大量用到了 Python 语言，同时看了一下 [TIOBE](https://www.tiobe.com/tiobe-index/） 上的指数，Python 最近今年的排名已经上升到和 C/C++，Java 并列的程度了，在这这篇文章时(2024 年 10 月)，Python 已经排在了第一位，前五名如下：

1. python
2. C++
3. Java
4. C
5. C#

TIOBE中是这么分析的：

> TIOBE Index for October 2024

> October Headline: Rust is slowly but steadily approaching the TIOBE index top 10

> In today's world, the amount of available data of whatever kind is increasing rapidly, and the demand to harvest this data is increasing accordingly. Hence, there is now a need for programming languages that are good in data manipulation, number crunching and being fast. Next to this, there are two other important characteristics high on everybody's list: languages should be easy to learn and should be secure. "Easy to learn" because the resource pool of skilled software engineers is drying up and "secure" because of continuous cyber threats. Languages that have these three traits (being fast, being secure and easy to learn), have a good time now.

> **King of all, Python, is easy to learn and secure, but not fast**. Hence, engineers are frantically looking for fast alternatives for Python. C++ is an obvious candidate, but it is considered "not secure" because of its explicit memory management. Rust is another candidate, although not easy to learn. Rust is, thanks to its emphasis on security and speed, making its way to the TIOBE index top 10 now.

Python 易于学习和掌握，而且比较安全，但是不够快。上述这些特点已经让 Python 可以应付很多使用场景了。是不是够快可能也不是使用者能左右的，因为这是编程语言本身实现决定的，就像 C++ 虽然没有 C 快，但是这也不妨碍 C++ 的流行。