---
layout: post
title:  "SONiC 中的 Python 编程"
date:   2024-10-14
category: programming
tags:   python sonic click tiobe
---

工作这些年的主力开发语言是 C/C++，确切地说是以 C 语言为主。最近一些年由于在 Linux 下开发，Shell 脚本语言用的也多了一些，各种编译构建，服务加载，系统监控等都可以通过 Shell 来实现。除此之外，在设备脚本执行，自动化测试等场景也涉及到了 Python，但是相对于 C 和 Shell，使用量还是少很多。

最近在深入研究交换机开源操作系统 SONiC，发现其中大量用到了 Python 语言，同时看了一下 [TIOBE](https://www.tiobe.com/tiobe-index/)上的指数，Python 最近几年的排名已经上升到和 C/C++，Java 并列的程度了，在这这篇文章时(2024 年 10 月)，Python 已经排在了第一位，前五名如下：

1. python
2. C++
3. Java
4. C
5. C#

TIOBE中是这么分析的：

> TIOBE Index for October 2024

> October Headline: Rust is slowly but steadily approaching the TIOBE index top 10

> In today's world, the amount of available data of whatever kind is increasing rapidly, and the demand to harvest this data is increasing accordingly. Hence, there is now a need for programming languages that are good in data manipulation, number crunching and being fast. Next to this, there are two other important characteristics high on everybody's list: languages should be easy to learn and should be secure. "Easy to learn" because the resource pool of skilled software engineers is drying up and "secure" because of continuous cyber threats. Languages that have these three traits (being fast, being secure and easy to learn), have a good time now.

> **King of all, Python, is easy to learn and secure, but not fast**. Hence, engineers are frantically looking for fast alternatives for Python. C++ is an obvious candidate, but it is considered "not secure" because of its explicit memory management. Rust is another candidate, although not easy to learn. Rust is, thanks to its emphasis on security and speed, making its way to the TIOBE index top 10 now.

Python 易于学习和掌握，而且比较安全，但是不够快。上述这些特点已经让 Python 可以应付很多使用场景了。是不是够快可能也不是使用者能左右的，因为这是编程语言本身实现决定的，就像 C++ 虽然没有 C 快，但是这也不妨碍 C++ 的流行。

其实 Python 是很全面的通用语言，除了丰富的数据处理库之外，操作系统中文件系统访问，网络通信接口，加密解密实现等等都有成熟的实现，Python 的跨平台支持让它不受限于一种操作系统，大量的功能库让它可以发挥更强大的功能。

比如SONiC中的命令行功能实现就依赖于 Python 的[Click](https://palletsprojects.com/projects/click/)，因为命令行是网络设备中最传统的用户交互方式，已经有相当多的成熟实现，比如最经典的Cisco IOS命令行风格，开源路由协议Frrouting就是模仿的这种风格。SONiC 使用的Click是通过 Python实现的命令行功能，其官网介绍如下：

> Click is a Python package for creating beautiful command line interfaces in a composable way with as little code as necessary. It's the "Command Line Interface Creation Kit". It's highly configurable but comes with sensible defaults out of the box.

> It aims to make the process of writing command line tools quick and fun while also preventing any frustration caused by the inability to implement an intended CLI API.

> Click in three points:
> 1. Arbitrary nesting of commands
> 2. Automatic help page generation
> 3. Supports lazy loading of subcommands at runtime

可见 Click 是一种 Python 实现的简单通用命令行工具，比 C 语言中的 getopt 功能更为强大。一个简单的示例程序如下：

```python
import click

@click.command()
@click.option('--count', default=1, help='Number of greetings.')
@click.option('--name', prompt='Your name',
              help='The person to greet.')
def hello(count, name):
    """Simple program that greets NAME for a total of COUNT times."""
    for x in range(count):
        click.echo(f"Hello {name}!")

if __name__ == '__main__':
    hello()
```

程序的执行结果如下
```shell
$ python hello.py --count=3
Your name: John
Hello John!
Hello John!
Hello John!
```

生成的帮助信息如下
```shell
$ python hello.py --help
Usage: hello.py [OPTIONS]

  Simple program that greets NAME for a total of COUNT times.

Options:
  --count INTEGER  Number of greetings.
  --name TEXT      The person to greet.
  --help           Show this message and exit.
```

可以看到 Click 实现的命令支持交互式和非交互式两种，分别适用于人机交互和脚本运行两种场景，而且帮助信息全面完整。Python 示例实现中还用到了装饰器 decorators 字符@，这让代码实现起来简单而强大。

除此之外，SONiC中还有大量组件使用了 Python，具体可以参考[sonic-buildimage](https://github.com/sonic-net/sonic-buildimage/tree/master/src)中的 src 目录。
