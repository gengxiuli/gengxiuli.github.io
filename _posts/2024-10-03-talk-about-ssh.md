---
layout: post
title:  "谈谈 ssh 的那些事儿"
date:   2024-10-03
category: networking
tags:   ssh
---

最近看到一篇文章[那些 ssh 教我的事](https://yangwenbo.com/articles/ssh-oh-my-god.html)，虽然是 2009 年发布的，虽然我也用了 ssh 很多年，但这些事儿我还真不知道，比如 ssh -L 和 ssh -R 这两个选项的作用，即使 SFTP 知道也用过，但是没有具体去研究原理，所以借着国庆休假的时间，谈谈 ssh 的那些事儿。

ssh 最广为人知的用处就是远程登录了，它比 telnet 更安全，因此现在绝大部分网络设备和服务器都支持 ssh 服务器，允许通过 ssh 客户端登录管理。我们使用这种方式登录服务器的时候，一般都会看到一些提示窗口，询问我们是否保存服务器密钥指纹(key fingerprint)，确认之后我们才能够使用用户名和密码登录服务器。[什么是SSH？](https://info.support.huawei.com/info-finder/encyclopedia/zh/SSH.html)这篇文章详细介绍了 ssh 的工作原理，这里就不再一一赘述了，需要注意的几点是：
1. 算法协商阶段涉及各种类型的算法，如果存在客户端和服务器对某个类型的算法达不成一致。则无法建立回话，这也是某些登录失败的原因。
2. 密钥交换过程中涉及对称加密和非对称加密，其中对称加密密钥用于后续的数据传输，而非对称加密用于回话建立阶段报文的交互。对称加密的密钥不是通过回话传输的，而是客户端和服务器分别计算出来的，这在数学上是可以保证二者是一样的。这样的好处是非对称密钥和对称密钥的私钥都不经过传输，保证了通信的安全。
2. 用户认证除了密码认证，还有密钥认证方式。因为平时主要用密码认证，对于密钥认证不是很了解，其实这是一种很重要的方式，因为密码认证虽然方便，但存在被暴力破解的可能。下面还会介绍一种证书登录方式，其实有些类似于 https 证书认证流程了。

[SSH 证书登录教程](https://www.ruanyifeng.com/blog/2020/07/ssh-certificate.html)这篇文章介绍了如何配置证书登录方式，其中对于证书登录有如下介绍：

> 证书登录就是为了解决上面的缺点而设计的。它引入了一个证书颁发机构（Certificate1 authority，简称 CA），对信任的服务器颁发服务器证书，对信任的用户颁发用户证书。

> 登录时，用户和服务器不需要提前知道彼此的公钥，只需要交换各自的证书，验证是否可信即可。

> 证书登录的主要优点有两个：（1）用户和服务器不用交换公钥，这更容易管理，也具有更好的可扩展性。（2）证书可以设置到期时间，而公钥没有到期时间。针对不同的情况，可以设置有效期很短的证书，进一步提高安全性。

本文最开始提到的 SFTP，其实也是利用 SSH 提供的一种文件传输服务，它比 FTP 更安全。Wiki 上的介绍如下：

> Compared to the SCP protocol, which only allows file transfers, the SFTP protocol allows for a range of operations on remote files which make it more like a remote file system protocol. An SFTP client's extra capabilities include resuming interrupted transfers, directory listings, and remote file removal. There is also support for all UNIX file types, including symbolic links.

> SFTP is not FTP run over SSH, but rather a new protocol designed from the ground up by the IETF SECSH working group. It is sometimes confused with Simple File Transfer Protocol.

SFTP 与 SCP 协议比较起来有很多优点，注意这里并没有 与 FTP 比较，因为 SFTP 是重新设计的协议，并不是 FTP over SSH，而 SCP 是一种安全的文件拷贝协议，Wiki 介绍如下：

> Secure copy protocol (SCP) is a means of securely transferring computer files between a local host and a remote host or between two remote hosts. It is based on the Secure Shell (SSH) protocol. "SCP" commonly refers to both the Secure Copy Protocol and the program itself.

但是 SCP 已经逐渐被 SFTP 或者 rsync 协议所取代：

> According to OpenSSH developers in April 2019, SCP is outdated, inflexible and not readily fixed; they recommend the use of more modern protocols like SFTP and rsync for file transfer. As of OpenSSH version 9.0, scp client therefore uses SFTP for file transfers by default instead of the legacy SCP/RCP protocol.

SSH 客户端软件[MobaXterm](https://mobaxterm.mobatek.net)在和服务器建立 SSH 会话后，在服务器支持 SFTP 功能后，会自动建立 SFTP 会话窗口，便于上传下载文件，功能非常实用。

除了 SFTP 外，网络设备上常用的 NETCONF 服务也支持 over 到 SSH 上，具体可以看 juniper 网站上的这篇文章：[Establish an SSH Connection for a NETCONF Session](https://www.juniper.net/documentation/us/en/software/junos/netconf/topics/topic-map/netconf-ssh-connection.html)，以及相关的 [rfc6242 Using the NETCONF Protocol over Secure Shell (SSH)](https://datatracker.ietf.org/doc/html/rfc6242)。

本文提到了 ssh 和 SSH 两种写法，其实有些细微差异。ssh 一般指客户端或者服务器上的 ssh 命令行，最广为人知的就是 openssh 开源软件了，他支持 SSH 客户端和服务器功能，不仅仅是 Linux 和 Mac 上默认的 SSH 客户端服务器程序，也成为了 Windows 10 之后的默认 SSH 工具，具体参考这里[OpenSSH for Windows overview](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh-overview)。

另外，我们常用的 github/gitlab 等 git 服务，除了通过 HTTPS 地址 clone 仓库在，也支持通过 SSH 方式 clone 仓库，如果你还需要修改上传代码，那只能使用 SSH 方式。但是和一般的通过 SSH 登录服务器方式不同，git 账户需要预先通过本地生成非对称加密的公钥和私钥，并把公钥指纹配置到 git 服务器上，后续基于此完成 git 服务的认证和使用。

而 SSH 是指由 IETF 定义的一种标准协议，全称是The Secure Shell (SSH)，它有 SSH 1 和 SSH 2 两个版本，现在主要用的都是 SSH 2。
