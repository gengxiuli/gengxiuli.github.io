---
layout: post
title:  "Linux ioctl cmd"
date:   2025-07-20
category: linux
tags:   ioctl
---

ioctl在linux上一种很传统的用户态与内核态的通信方式，它不像netlink方式那样支持双向通信，也不如proc那样，可以让用户在shell下通过cat或者echo直接和内核通信，但是它有很好的兼容性，所以还具有很广泛的使用，比如一些轻量级的用户态和内核通信。

tcp/ip网络协议栈的arp功能就支持通过ioctl添加静态/删除静态arp，内核中的实现在arp.c文件中的arp_ioctl接口中，其最终又是通过af_inet.c中的inet_ioctl来调用的。在inet_ioctl中可以看到，其通过判断cmd来决定调用哪个模块，上面的arp功能对应用户态态的arp命令。除此之外，dev_ioctl.c中的dev_ioctl也实现了设备支持的很多iotcl命令，这些功能对应用户态的ifconfig命令。所有网络层使用的ioctl中对应的cmd命令字都可以在[sockios.h](https://elixir.bootlin.com/linux/v5.10.70/source/include/uapi/linux/sockios.h)中找到，其中包括Socket configuration controls，ARP cache control calls，bonding calls，bridge calls等等，设备还可以自定义私有的ioctl命令字，内核保留了0x89F0-0x89FF和0x89E0-0x89EF两个段，每段支持16个命令字。

其实除了上面内核原生支持的ioctl命令字外，内核还支持自定义字符设备，而字符设备又可以基于每个设备实现自己的ioctl命令，其中的cmd是该设备私有的，不存在cmd与其他模块的冲突问题，很好的支持了用户态程序和内核态模块的通信需求。

内核对于ioctl命令字实现了一系列的预定义宏，这样模块自己可以根据需要实现READ/WRITE/READ AND WRITE操作，支持根据type区分不同的命令，支持命令数据长度size, 以及相同type下的不同nr索引，这些宏定义在文件[ioctl.h](https://elixir.bootlin.com/linux/v5.10.70/source/include/uapi/asm-generic/ioctl.h)中。注意这里面的READ和WRITE是从用户态视角看的，ioctl.h中对此也做了说明。

```c
/*
 * Used to create numbers.
 *
 * NOTE: _IOW means userland is writing and kernel is reading. _IOR
 * means userland is reading and kernel is writing.
 */
#define _IO(type,nr)		_IOC(_IOC_NONE,(type),(nr),0)
#define _IOR(type,nr,size)	_IOC(_IOC_READ,(type),(nr),(_IOC_TYPECHECK(size)))
#define _IOW(type,nr,size)	_IOC(_IOC_WRITE,(type),(nr),(_IOC_TYPECHECK(size)))
#define _IOWR(type,nr,size)	_IOC(_IOC_READ|_IOC_WRITE,(type),(nr),(_IOC_TYPECHECK(size)))
#define _IOR_BAD(type,nr,size)	_IOC(_IOC_READ,(type),(nr),sizeof(size))
#define _IOW_BAD(type,nr,size)	_IOC(_IOC_WRITE,(type),(nr),sizeof(size))
#define _IOWR_BAD(type,nr,size)	_IOC(_IOC_READ|_IOC_WRITE,(type),(nr),sizeof(size))
```

其中_IOC定义如下:

```c
#define _IOC(dir,type,nr,size) \
	(((dir)  << _IOC_DIRSHIFT) | \
	 ((type) << _IOC_TYPESHIFT) | \
	 ((nr)   << _IOC_NRSHIFT) | \
	 ((size) << _IOC_SIZESHIFT))
```

方向位有预定义设置，但是不同架构也可以定义自己的实现，内核的默认设置如下：

```c
/*
 * Direction bits, which any architecture can choose to override
 * before including this file.
 *
 * NOTE: _IOC_WRITE means userland is writing and kernel is
 * reading. _IOC_READ means userland is reading and kernel is writing.
 */

#ifndef _IOC_NONE
# define _IOC_NONE	0U
#endif

#ifndef _IOC_WRITE
# define _IOC_WRITE	1U
#endif

#ifndef _IOC_READ
# define _IOC_READ	2U
#endif
```

type, nr和size都是ioctl自己定义的，内核只实现了按bit来区分不同的段，具体如下：

```c
#define _IOC_NRMASK	((1 << _IOC_NRBITS)-1)
#define _IOC_TYPEMASK	((1 << _IOC_TYPEBITS)-1)
#define _IOC_SIZEMASK	((1 << _IOC_SIZEBITS)-1)
#define _IOC_DIRMASK	((1 << _IOC_DIRBITS)-1)

#define _IOC_NRSHIFT	0
#define _IOC_TYPESHIFT	(_IOC_NRSHIFT+_IOC_NRBITS)
#define _IOC_SIZESHIFT	(_IOC_TYPESHIFT+_IOC_TYPEBITS)
#define _IOC_DIRSHIFT	(_IOC_SIZESHIFT+_IOC_SIZEBITS)
```

size和dir字段支持架构自定义,如果没有定义则默认如下：

```c
/*
 * Let any architecture override either of the following before
 * including this file.
 */

#ifndef _IOC_SIZEBITS
# define _IOC_SIZEBITS	14
#endif

#ifndef _IOC_DIRBITS
# define _IOC_DIRBITS	2
#endif
```

nr和type字段的长度内核是固定设置好的：

```c
/*
 * The following is for compatibility across the various Linux
 * platforms.  The generic ioctl numbering scheme doesn't really enforce
 * a type field.  De facto, however, the top 8 bits of the lower 16
 * bits are indeed used as a type field, so we might just as well make
 * this explicit here.  Please be sure to use the decoding macros
 * below from now on.
 */
#define _IOC_NRBITS	8
#define _IOC_TYPEBITS	8
```

所以，ioctl中的cmd是一个32bit的数字，具体含义内核解释如下：

```c
/* ioctl command encoding: 32 bits total, command in lower 16 bits,
 * size of the parameter structure in the lower 14 bits of the
 * upper 16 bits.
 * Encoding the size of the parameter structure in the ioctl request
 * is useful for catching programs compiled with old versions
 * and to avoid overwriting user space outside the user buffer area.
 * The highest 2 bits are reserved for indicating the ``access mode''.
 * NOTE: This limits the max parameter size to 16kB -1 !
 */
 ```

 综上所述，内核ioctl中cmd的默认bit按段排列如下：

 31 - 30 (2bit) :   Direction

 29 - 16 (14bit):   Size

 15 - 8  (8bit) :   Type

 7 - 0   (8bit) :   NR

                                


