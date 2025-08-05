---
layout: post
title:  "WSL linux kernel ko build"
date:   2025-08-03
category: linux
tags:   kernel ko
---

在linux下编译ko模块，以便在内核中增加自定义的驱动或者协议，是内核开发中很重要的一部分。WSL目前在Windows上基本可以代替各种虚拟机方案了，使用起来简单而又强大。

但是WSL本身没有提供Linux Kernel对应的头文件，所以无法直接编译ko模块，需要做一些工作才能完成ko模块的编译和验证。

1. 使用uname -r查看WSL对应的内核版本，然后在[https://github.com/microsoft/WSL2-Linux-Kernel/releases](https://github.com/microsoft/WSL2-Linux-Kernel/releases)找对应的内核版本。如果你想尝试比较新的内核版本也可以，比如验证新的内核特性。下载对应的内核tar.gz文件到本地解压。

2. 在WSL下安装必要的编译工具，一般的是：sudo apt install build-essential flex bison dwarves libssl-dev libelf-dev

3. 进入上面第1步解压出的内核目录，执行cp Microsoft/config-wsl .config，以便拷贝Microsoft提供的内核配置文件，然后分别执行make scripts和make modules，前者用来构建编译环境，后者进行kernel的实际编译工作[这里其实应该执行的是make而不是make modules，后面会有解释]。如果一切正常，你会在内核目录生成vmlinuz文件。

4. 编写ko模块，并编写Makefile文件，其中KDIR指向内核目录，完成后执行make编译ko模块。

5. 如果ko模块代码没什么错误，最后在当前目录会生成foo.ko文件，使用sudo insmod foo.ko将ko模块加载到系统中。

6. 如果第5步提示类似下面的错误，是因为make modules只编译了.config中的=M模块，没有完整编译内核且生成bzImage，下面BPF[或者BTF]加载失败问题应该和ko和内核版本不匹配有关。所以需要用make重新编译内核，然后更新wsl的内核后重启wsl,再安装自己编写的ko就可以了

```shell
[17581.503703] BPF:[133261] Invalid name_offset:2373844
[17581.504935] failed to validate module [hello] BTF: -22
```

7. "需要用make重新编译内核，然后更新wsl的内核后重启wsl",具体是指在kenel目录下执行make,然后将生成的文件arch/x86_64/boot/bzImage拷贝到Windows文件系统的一个目录下，比如/mnt/c/Users/abc，其中abc是用户名称,也可以是其他任何目录。

```shell
cp arch/x86_64/boot/bzImage /mnt/c/Users/abc
```

然后在该目录下创建或者修改.wslconfig文件，添加下面的内容，注意路径中的\是两个，而且必须是Windows的文件路径：

```shell
[wsl2]
kernel=C:\\Users\\abc\\bzImage
```

完成后在Windows命令行下执行wsl --shutdown关闭WSL，然后再次开启，成功进入系统后用uname -a查看版本是否更新成功。