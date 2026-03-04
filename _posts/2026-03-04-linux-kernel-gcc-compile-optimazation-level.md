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

上面提到：By default, most of the kernel is build with C's -O2,参考的是<https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Makefile#n892>，注意写作本文时Linux内核主线版本是7.0-rc2，所以更新了一下这里Makefile对应的行号。具体Malefile内容如下。

```
ifdef CONFIG_CC_OPTIMIZE_FOR_PERFORMANCE
KBUILD_CFLAGS += -O2
KBUILD_RUSTFLAGS += -Copt-level=2
else ifdef CONFIG_CC_OPTIMIZE_FOR_SIZE
KBUILD_CFLAGS += -Os
KBUILD_RUSTFLAGS += -Copt-level=s
endif
```

可以看出，如果定义了宏CONFIG_CC_OPTIMIZE_FOR_PERFORMANCE则使用O2优化编译，如果定义了CONFIG_CC_OPTIMIZE_FOR_SIZE则使用Os优化编译，那么如何看出默认是哪钟编译呢？

在文件<https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/init/Kconfig>中，我们可以看到下面的内容：

```

choice
	prompt "Compiler optimization level"
	default CC_OPTIMIZE_FOR_PERFORMANCE

config CC_OPTIMIZE_FOR_PERFORMANCE
	bool "Optimize for performance (-O2)"
	help
	  This is the default optimization level for the kernel, building
	  with the "-O2" compiler flag for best performance and most
	  helpful compile-time warnings.

config CC_OPTIMIZE_FOR_SIZE
	bool "Optimize for size (-Os)"
	help
	  Choosing this option will pass "-Os" to your compiler resulting
	  in a smaller kernel.

endchoice
```

可以看出默认的选项就是CC_OPTIMIZE_FOR_PERFORMANCE。