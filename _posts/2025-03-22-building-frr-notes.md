---
layout: post
title:  "Building FRR Notes"
date:   2025-03-22
category:  networking
tags:   frr sonic
---

最近在一台 Ubuntu 上编译 frr, 发现编译之后运行总是提示找不到路径 /etc/frr.conf，而改路径确实没有这个文件，frr.conf 的路径是 /etc/frr。
后来看代码发现当前编译的版本是9.1.3，而之前按照同样的方法编译的是10.0.1。难道和版本有关，经过一番分析和查找，终于发现 frr 官网上[Ubuntu22.04
的构建方法](https://docs.frrouting.org/projects/dev-guide/en/latest/building-frr-for-ubuntu2204.html)是针对最新的版本(写作本文时是10.3)。
在执行 ./configure 时配置的是：

```shell
    --sysconfdir=/etc \
    --localstatedir=/var \
```

而在9.2版本之前，当然也包括9.1.3，在执行 ./configure 时配置的是

```shell
    --sysconfdir=/etc/frr \
    --localstatedir=/var/run/frr
```

而原因在最新代码的 [configure.ac](https://github.com/FRRouting/frr/blob/master/configure.ac)中已经给出了说明：

> ------------------------------
> system paths
> ------------------------------
> Versions of FRR (or Quagga, or Zebra) before ca. 9.2 used sysconfdir and
> localstatedir as-is, without appending /frr.  The /frr was expected to be
> given on ./configure invocations.
> 
> This does not match standard behavior by other packages and makes FRR
> specific packaging changes necessary to add these options.  localstatedir
> was also misused to include the /run part (it normally is only /var),
> leaving no path configuration option that references /var itself.  This
> is because runstatedir did not exist in ancient autoconf.
> 
> The path options have been changed to expect plain / system prefix
> directories.  As a temporary workaround to not break packaging, eventual
> /frr suffixes are stripped and a warning is printed.

简单来说就是大概在9.2版本之前的 sysconfdir 和 localstatedir，需要手动在执行 ./configure 时候配置 frr 完整路径，而这种方式和其他包使用的标准方式不同。
另外为了兼容性的考虑，如果用户按照之前的方式配置 configure， 则脚本中会自动去除末尾的 /frr，保证 sysconfdir 和 localstatedir 只包含定义目录，这通常
是指路径 /etc 和 /var。

查看 configure.ac 文件的修改历史，发现这个 [merge](https://github.com/FRRouting/frr/commit/ff62df2e4484b9f89fea4ed736006c21f3a797cc) 是在 2024年1月28日合入的。commit信息如下：

> build: untangle sysconfdir & localstatedir
> `--sysconfdir` should be `/etc` and `--localstatedir` should be `/var`.
> The package-specific subdirectory should be added by configure, not
> given by the user, to match established behavior by other packages.
> 
> Note that `--bindir`, `--sbindir`, `--libdir` and `--libexecdir` have
> different established/expected behavior due to distro specific
> multi-arch support.  That's why these are left unchanged.
> 
> The reason this is getting fixed now is that we need to use
> `--localstatedir` for its actual value to put things in `/var/lib`.  As
> it is now, being overloaded for `/run`, the configured `/var` path
> becomes inaccessible.
> 
> Signed-off-by: David Lamparter <equinox@opensourcerouting.org>

综上所述，使用 9.2 之前的 configure 配置可以在所有版本上正常编译安装 frr，因为脚本考虑了兼容性。但是使用最新的 configure 编译指导文档去配置 9.2 之前的 frr 代码，则会出现上面提到的 frr 运行时找不到 /etc/frr.conf文件的问题。这其实也时一个兼容性问题，因为旧代码改不了了，所以最新的文档最好将这点说明一下。

