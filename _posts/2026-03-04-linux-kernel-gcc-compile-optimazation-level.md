---
layout: post
title:  "Linux kernel gcc compile optimization level"
date:   2026-03-04
category: linux
tags:   gcc
---

wiki.gentoo.org 上的这篇文章[Kernel/Optimization](https://wiki.gentoo.org/wiki/Kernel/Optimization)介绍了很多关于 Linux 内核编译优化的事情。其中有如下一段。

> *FLAGS

> Note

> For more list of *FLAGS to play with, see the GCC manual and Clang manual or man 1 gcc and man 1 clang commands.
By default, most of the kernel is build with C's -O2 (some code, like Random Number Generation, does not work with optimizations and sometimes checked with the C macro __OPTIMIZE__). This can be changed via Kbuild. Before making any KCFLAGS and similar flags, please check the kernel's Makefiles before it gets any changes. For example, -fallow-store-data-races is disabled on this Makefile.

> -O3

> The command to add this flag is:
> make KCFLAGS="-O3"

> There was a official attempt to add -O3 to the kernel but Linus Torvalds reject it due to -O3 historically outputting worse code than -O2. Phoronix ran a -O3 kernel benchmark and found nearly all tested programs to have no measurable benefit.