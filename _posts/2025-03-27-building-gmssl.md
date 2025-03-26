---
layout: post
title:  "Building GmSSL"
date:   2025-03-27
category:  networking
tags:   gmssl
---

GmSSL是一个开源的密码工具箱，支持SM2/SM3/SM4/SM9/ZUC等国密(国家商用密码)算法、SM2国密数字证书及基于SM2证书的SSL/TLS安全通信协议，支持国密硬件密码设备，提供符合国密规范的编程接口与命令行工具，可以用于构建PKI/CA、安全通信、数据加密等符合国密标准的安全应用。GmSSL项目是OpenSSL项目的分支，并与OpenSSL保持接口兼容。因此GmSSL可以替代应用中的OpenSSL组件，并使应用自动具备基于国密的安全能力。GmSSL项目采用对商业应用友好的类BSD开源许可证，开源且可以用于闭源的商业应用

1. 下载源代码 

```shell
git clone https://github.com/guanzhi/GmSSL.git
```

2. 编译构建

GmSSL 3 采用了cmake构建系统。下载源代码后将其解压缩，进入源码目录，执行：

```shell
mkdir build
cd build
cmake ..
make
make test
sudo make install
```

在make install完成后，GmSSL会在默认安装目录中安装gmssl命令行工具，在头文件目录中创建gmssl目录，并且在库目录中安装libgmssl.a、libgmssl.so等库文件。

3. 交叉编译ARM64平台的版本

安装交叉编译工具链

```shell
sudo apt install gcc-aarch64-linux-gnu
```

创建cmake文件arm64.toolchain.cmake,内容如下：

```shell
# the name of the target operating system
set(CMAKE_SYSTEM_NAME linux)

# which compilers to use for C and C++
set(CMAKE_C_COMPILER   aarch64-linux-gnu-gcc)
set(CMAKE_CXX_COMPILER aarch64-linux-gnu-g++)

# search headers and libraries in the target environment
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```

执行如下编译命令

```shell
mkdir build; cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE=arm64.toolchain.cmake
make
```

查看编译出的gmssl可执行文件信息,其中Machine字段中的AArch64就是ARM64版本。

```shell
readelf -h bin/gmssl

ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           AArch64
  Version:                           0x1
  Entry point address:               0x77c0
  Start of program headers:          64 (bytes into file)
  Start of section headers:          278312 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         9
  Size of section headers:           64 (bytes)
  Number of section headers:         28
  Section header string table index: 27
```

参考：
https://cmake.org/cmake/help/book/mastering-cmake/chapter/Cross%20Compiling%20With%20CMake.html