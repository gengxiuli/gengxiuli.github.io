---
layout: post
title:  "Linux memory usage"
date:   2025-06-22
category:  programming
tags:   linux memory
---

因为需要计算 Linux 下的内存使用率，搜索了一下发现目前主要的计算方法是：

```c
Usage = (MemTotal - Available) / MemTotal * 100%
```
其中 MemTotal 和 Available 通过 free 或者 cat /proc/meminfo 可以获取到。

free 的输出：
```shell
               total        used        free      shared  buff/cache   available
Mem:         7778640      813616     6749036        3192      452044     6965024
Swap:        2097152           0     2097152
```
cat /proc/meminfo 的输出(部分)
```shell
MemTotal:        7778640 kB
MemFree:         6748432 kB
MemAvailable:    6964420 kB
Buffers:           33192 kB
Cached:           382928 kB
SwapCached:            0 kB
Active:           177664 kB
Inactive:         399212 kB
Active(anon):       2524 kB
Inactive(anon):   161424 kB
Active(file):     175140 kB
Inactive(file):   237788 kB
Unevictable:           0 kB
Mlocked:               0 kB
```
注意，上述 available/MemAvailable并不是通过 free + buff/cache 简单计算得到的，而是在内核中通过比较复杂的算法得到的。
其最开始合入内核的commit可以参考这里：[/proc/meminfo: provide estimated available memory](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=34e431b0ae398fc54ea69ff85ec700722c9da773)，合入主线的时间是2014年1月21日，对应的内核版本是[3.14](https://kernelnewbies.org/Linux_3.14),对应的文件是[meminfo.c](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/diff/fs/proc/meminfo.c?id=34e431b0ae398fc54ea69ff85ec700722c9da773)。其合入的commit信息如下：

```
Many load balancing and workload placing programs check /proc/meminfo to
estimate how much free memory is available.  They generally do this by
adding up "free" and "cached", which was fine ten years ago, but is
pretty much guaranteed to be wrong today.

It is wrong because Cached includes memory that is not freeable as page
cache, for example shared memory segments, tmpfs, and ramfs, and it does
not include reclaimable slab memory, which can take up a large fraction
of system memory on mostly idle systems with lots of files.

Currently, the amount of memory that is available for a new workload,
without pushing the system into swap, can be estimated from MemFree,
Active(file), Inactive(file), and SReclaimable, as well as the "low"
watermarks from /proc/zoneinfo.

However, this may change in the future, and user space really should not
be expected to know kernel internals to come up with an estimate for the
amount of free memory.

It is more convenient to provide such an estimate in /proc/meminfo.  If
things change in the future, we only have to change it in one place.
```
最新版本的内核(6.16),上述avaiable的计算已经放到[show_mem.c](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/mm/show_mem.c)中了。

这篇文章[我的内存呢？Linux MemAvailable 如何计算](https://lotabout.me/2021/Linux-Available-Memory/)介绍了MemAvailable的计算方法，有兴趣可以自己尝试计算一下。文中还提到另外一个问题，我们如何计算每个进程(或者进程中每个线程)的内存使用情况呢？如果一个进程包含了很多线程，可以通过 top 命令中的 thread 视图查看(大写的H)，但是如何通过程序计算出来，可能需要我们代码去实现了。另外，将一个比较大的程序拆分为多个进程，其实天生就支持了多进程内存查看，比多线程要更加直观一些,这也算是多进程的一个优点了吧。