---
layout: post
title:  "Python 数据结构代码示例(四)"
date:   2024-10-25
category: programming
tags:   python counter deque defaultdict
---

除了之前介绍的几种常用数据结构外，Python的collections模块还支持了很多容器数据类型(Container datatypes),研究了一下，其中counter，deque和defaultdict看起来比较有用，特此记录一下。

### 1.counter

Counter是Dict的一个子类(subclass),主要用于可散列对象的计数场景。具体数据结构实现是Dict，其中key存储需要计数的对象，而value则是该对象对应的计数，可以认为是一种特殊的Dict字典。
Counter的value可以是任何整型值包括0和负数，这里的负数有点意思，什么场景统计需要负数呢？从数学角度说，负数和正数只是方向的差异，从财务角度负数就是欠账，总之这种设计可以覆盖更多的使用场景。
统计值0和负值都只是value的不同数值，对应的key还是存在的，也就是Dict中有一条记录。另外，如果使用c[key]的方式访问Counter，如果key不存在则返回也是0，但是和上面的0其实是不一样的，后面这种Dict中是没有对应的记录的。

### 2.deque

### 3.defaultdict


参考资料
1. <https://docs.python.org/3/library/collections.html>
2. <https://docs.python.org/3/library/functions.html#sorted>
3. <https://github.com/python/cpython/blob/3.13/Lib/collections/__init__.py>
4. <https://github.com/python/cpython/blob/main/Modules/_collectionsmodule.c>
5. <https://docs.python.org/3/library/operator.html>
