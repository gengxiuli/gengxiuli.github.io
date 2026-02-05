---
layout: post
title:  "Frrouting bgp debug"
date:   2026-02-06
category: networking
tags:   Frrouting bgp
---

打开调试keepalive开关(enable模式)
debug bgp keepalives

打开log到File功能(config模式)
log file /var/log/frr/debug.log

查看log文件
/var/log/frr$ ls -l
total 8
-rw-r----- 1 frr frr 7786 Feb  6 06:47 debug.log

关闭调试keepalive开关(enable模式)
no debug bgp keepalives

关闭log到File功能(config模式)
no log file

查看debug调试开关状态
show debugging

查看log状态
show logging

参考资料：
<https://docs.nvidia.com/networking-ethernet-software/cumulus-linux-41/Layer-3/Configuring-FRRouting/>
<https://docs.frrouting.org/projects/dev-guide/en/latest/logging.html>
<https://docs.frrouting.org/en/latest/basic.html#clicmd-log-file-FILENAME-LEVEL>


