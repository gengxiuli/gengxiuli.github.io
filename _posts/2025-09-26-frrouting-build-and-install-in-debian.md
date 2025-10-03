---
layout: post
title:  "Frrouting build and install in debian"
date:   2025-09-26
category: networking
tags:   debian frrouting
---

在Debian 12下用源码编译和安装Frrouting，记录一下遇到的问题和解决方法。

1. 官方编译指导： <https://docs.frrouting.org/projects/dev-guide/en/latest/building-frr-for-debian12.html>

2. libyang 在Debian下用apt安装后的版本小于2.1.128，因此在./configure的时候会提示版本过小，需要安装2.1.128及以后的版本。
   
   我这里采用的是deb包安装，直接在 <https://ci1.netdef.org/browse/LIBYANG-LIBYANG21-15/artifact/shared/Debian-12-x86_64-Packages/> 下载deb文件即可。需要注意的是，如果安装dev版本libyang2-dev_2.1.128-2~deb12u2_amd64.deb，则必须要安装libyang2_2.1.128-2~deb12u2_amd64.deb，否则只安装后者就可以。安装命令：sudo dpkg -i xxx.deb。

3. 官方编译指导 <https://docs.frrouting.org/projects/dev-guide/en/latest/building-frr-for-debian12.html> 中在编译及安装配置文件之后，并没有给如何安装服务的方法。参考 Debian 13 或者 Ubuntu 24.04 的安装服务方法，在编译完成后可以执行如下命令安装服务：

   ```shell
   sudo install -m 644 tools/frr.service /etc/systemd/system/frr.service
   sudo systemctl enable frr
   ```

4. 当使用 systemctl start frr 启动服务的时候，可能会提示如下错误：

    Job for frr.service failed because the service did not take the steps required by its unit configuration.
    See "systemctl status frr.service" and "journalctl -xeu frr.service" for details.

    这个错误看不出来什么问题，可以通过 journalctl -xeu frr.service 查看错误日志分析。

    我这里的错误日志如下：

    ```
    A start job for unit frr.service has begun execution.

    The job identifier is 5853226.
    
    Sep 26 06:20:44 VM-debian-60.205.136.106-aliyun frrinit.sh[1471783]: Starting watchfrr with command: '  /usr/lib/frr/watchfrr  -d   zebra mgmtd staticd'.
    Sep 26 06:20:44 VM-debian-60.205.136.106-aliyun frrinit.sh[1471792]: /usr/lib/frr/watchfrr: error while loading shared libraries: libfrr.so.0: cannot open shared object file: No s>
    Sep 26 06:20:44 VM-debian-60.205.136.106-aliyun frrinit.sh[1471783]: Failed to start watchfrr! ... failed!
    Sep 26 06:20:44 VM-debian-60.205.136.106-aliyun systemd[1]: frr.service: Can't open PID file /run/frr/watchfrr.pid (yet?) after start: No such file or directory
    Sep 26 06:20:44 VM-debian-60.205.136.106-aliyun systemd[1]: frr.service: Failed with result 'protocol'.

    Subject: Unit failed
    Defined-By: systemd
    Support: https://www.debian.org/support
    
    The unit frr.service has entered the 'failed' state with result 'protocol'.
    Sep 26 06:20:44 VM-debian-60.205.136.106-aliyun systemd[1]: Failed to start frr.service - FRRouting.
    Subject: A start job for unit frr.service has failed
    ```

    可以看出是启动watchfrr的时候无法找到libfrr.so.0引起的。
    
    官网其实已经给出了解决方法：在 /etc/ld.so.conf 中增加一行： echo include /usr/local/lib。

    修改完成执行ldconfig使上述修改立即生效，然后再次 systemctl start frr 启动服务即可。

5. 如果上述服务只启动了一部分协议，是因为 /etc/frr/daemons 中没有打开其他协议，编辑 /etc/frr/daemons， 将所有服务修改为 =yes，保存后 systemctl restart frr 重启服务。修改完成后的 /etc/frr/daemons 如下：

    ```shell
    bgpd=yes
    ospfd=yes
    ospf6d=yes
    ripd=yes
    ripngd=yes
    isisd=yes
    pimd=yes
    pim6d=yes
    ldpd=yes
    nhrpd=yes
    eigrpd=yes
    babeld=yes
    sharpd=yes
    pbrd=yes
    bfdd=yes
    fabricd=yes
    vrrpd=yes
    pathd=yes
    ```
6. 完成上述安装和服务启动后，就可以通过sudo vtysh登录 frr 来正常使用了。



