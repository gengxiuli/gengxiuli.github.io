---
layout: post
title:  "Linux kernel neighbour entry remove"
date:   2025-10-03
category: networking
tags:   linux neighbour
---

在解决Linux内核neighbour子系统问题的过程中发现，直到patch: [neigh: Really delete an arp/neigh entry on "ip neigh delete" or "arp -d"](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/commit/net/core/neighbour.c?id=5071034e4af709d6783b7d105dc296a5cc84739b)被合入，Linux才真正支持手动删除一个Neighbour表项，对应的支持版本是4.13。所以，在此之前的版本，用ip neigh delete或者arp -d都无法完全删除一个arp/neigh表项。

奇怪的是，手边正好有1个CentOS 7.8的云服务器,内核版本是3.10.0, 试了一下能够删除掉啊。最近研究了一下发现，这是因为CentOS backport(向前移植)了4.12之后代码，所以可以支持这个特性。

```shell
    $uname -a
    Linux VM-4-5-centos 3.10.0-1160.88.1.el7.x86_64 #1 SMP Tue Mar 7 15:41:52 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
```

确认方法如下：

在[https://vault.centos.org/7.9.2009/os/Source/SPackages/](https://vault.centos.org/7.9.2009/os/Source/SPackages/)找到内核代码压缩包：kernel-3.10.0-1160.el7.src.rpm，下载到本地，然后执行下面的操作获得压缩包：linux-3.10.0-1160.118.1.el7.tar.xz。

1. 创建mockbuild组和用户

```shell
    sudo groupadd mockbuild
    sudo useradd mockbuild -g mockbuild
```

2. 安装rpm文件

```shell
    rpm -ivh kernel-3.10.0-1160.118.1.el7.src.rpm
```

3. 找到压缩包

```shell
    cd ~/rpmbuild/SOURCES/
    tar Jxvf linux-3.10.0-1160.118.1.el7.tar.xz
```

至此，目录linux-3.10.0-1160.118.1.el7就是Linux内核源文件了，找到其中的net/core/neighbour.c，可以看到已经合入了 neigh_del， neigh_remove_one的实现。

我的CentOS 7.8使用的内核正常应该是3.10.0-1127(https://vault.centos.org/7.8.2003/os/Source/SPackages/kernel-3.10.0-1127.el7.src.rpm)，但是实际上是3.10.0-1160，这可能是云服务商已经默认把内核更新成最高版本了。