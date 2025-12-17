---
layout: post
title:  "Linux kernel version numbers"
date:   2025-12-17
category: linux
tags:   version
---

Linux Kernel稳定版本维护者Greg Kroah-Hartman在个人博客[Linux Kernel Monkey Log (https->http)](https://www.kroah.com/log/)上发表了一篇文章[Linux kernel version numbers(https->http)](https://www.kroah.com/log/blog/2025/12/09/linux-kernel-version-numbers/)。详细介绍了关于Linux Kernel的一些版本信息，包括版本命名规则，稳定版本发布流程，分支开发流程，稳定版本维护流程等。

关于Kernel Version需要记住的事情：

All releases are “stable” and backwards compatible for userspace programs to all previous kernel releases.<br>
所有发布版本都是稳定的，而且都是向后兼容所有依赖之前内核发布版本的用户空间程序。

Higher major and minor version numbers mean a newer release, and do not describe anything else.<br>
更高的主版本号和小版本号意味着是更新的发布版本，除此之外不意味着其他任何事情。

关于稳定版本内核分支：

The stable releases would take the bugfixes that went into the current development tree, and apply them to the previous stable release and do a new release that users could then use.<br>
稳定的发布版本会采纳已经合并到当前开发分支的缺陷修复，然后把他们合入之前的稳定发布分支，最后发布一个新的用户可以使用的发布版本。

The rules of what is acceptable into a stable kernel are documented with the most important rule being the first one “It or an equivalent fix must already exist in Linux mainline (upstream).”<br>
对于什么是可以接收合入稳定内核版本的首要原则是，也是最重要的原则，它或者相同的修复必须已经存在于Linux主线版本(上游版本)。

作者列举了一个5.2版本的例子来说明上述内核版本的发布流程。

![release_cycle](/assets/image/release_cycle.png)

First 5.2.0 was released by Linus, and then he continued on with the 5.3 development cycle, first releasing -rc1, and then -rc2, and so on until -rc7 which was followed by a stable release, 5.3.0.<br>
首先linus发布了5.2.0, 然后他继续5.3的开发周期。首先发布-rc1, 然后发布-rc2, 一直到-rc7，最后发布一个稳定版本5.3.0。

At the time 5.2.0 was released, it was branched in the Linux stable git tree by the stable kernel maintainers, and stable releases started happening, 5.2.1, 5.2.2, 5.2.3, and so on. The changes in these stable releases were all first in Linus’s tree, before they were allowed to be in a stable release, ensuring that when a user upgrades from 5.2 to 5.3, they will not have any regressions of bugfixes that might have only gone into the 5.2.stable releases.<br>
当5.2.0发布的时候，由稳定内核版本维护者从主线拉出对应的稳定版本分支到稳定版本仓库(译者注：这里是fork而不是checkout),稳定发布版本继续演进到5.2.1, 5.2.2, 5.2.3等等。在这些稳定发布版本分支上的修改，当他们被允许合入稳定发布版本之前，必须首先合入linus的开发主线，这样可以确保用户从5.2升级到5.3的时候，他们不会有任何的缺陷修复功能回退，也就是只在5.2稳定版本修复而没有在5.3稳定版本修复。

从上面可以看出，所有合入稳定版本的修改，必须先进入最新的主线，这样保证了主线代码是最全的特性以及缺陷修复。除了上面提到的之外，其实还有一些事情需要注意。

1.因为目前内核有多个稳定发布版本(截止到2025年12月17日，有5.10, 5.15, 6.1, 6.6, 6.12, 6.18共6个),具体可以参考<https://www.kernel.org/category/releases.html>。所以上面提到的主线缺陷修复合入稳定发布分支的工作，还需要在当前所有发布分支修复一遍。因为不同分支可能存在实现差异，上面的修复同步工作并不是那么简单。比如主线修复的问题在稳定版本并没有支持，或者稳定版本存在缺陷的代码在主线上已经修改或者彻底删除，这都需要根据具体问题具体修改，而不是直接把主线拿过来直接合入。从当前2年的维护周期来看，当2026年12月之后，5.10, 5.15, 6.6. 6.12这4个稳定版本都会完成维护周期，加上那个时候发布的版本（称为版本X1,EOL 2028年12月），就只剩下3个稳定版本了。而2027年12月之后,6.1和6.18也会完成维护周期而终止，加上同时发布的版本(称为版本X2, EOL 2029年12月)，则只需要维护2个版本了。自此之后，每年年底有1个版本EOL，同时发布一个新版本，内核稳定版本只需要维护2个了，z版本号也不会像5.4出现302这么大了，2年的时间大概100+左右。

2.内核这种缺陷修复先上主线，再上稳定发布版本策略，可以保证很好的兼容性。除此之外，还有一种策略是先在稳定版本分支上修复，再合入主线，这种看似效果一样，但是在稳定版本直接修复存在一定风险，而内核这种先在主线修复，验证没问题之后合入稳定版本，是一种更为稳健的策略。另外先合入稳定版本，后续忘记合入主线，就会主线稳定版本已经修复的问题，在新的主线或者发布分支又出现了版本回退。内核上述策略则肯定不会出现这个问题，因为是先在主线修复的。即使忘记合入发布分支，也只会导致新版本解决了而旧版本没解决。注意，如果存在多个稳定版本，则从主线同步的缺陷修复应该先合入更新的稳定版本，再合入之前的版本，以此类推，以保证不存在版本回退问题。

3.发布策略还有一种是在发布阶段直接从主线拉出预发布分支，发布分支只合入重要及严重问题，主线做下一阶段的特性开发。这种策略的好处是当产品存在多个代码库的时候，可以在发布之前先将版本稳定，然后针对性的修复。存在的可能问题是，分支上的缺陷修改要及时同步或者双合进主线，以防止出现版本退回问题。另外，所有在主线开发的人员，都要同时切换到稳定分支开发，以便完成发布版本工作。

4.内核的开发周期-rc1到-rc7不是完全一样的，-rc1是很多子系统合入新特性的地方，然后-rc2到-rc7是对这些特性的增强和修复，-rc7几乎都是一些修复工作了，如果有特殊则会延续到—rc8。与此同时，一些不稳定的新特性在继续下一轮的开发工作(从主线fork出自己的仓库)，直到稳定后申请合入当前主线，注意主线合入有merge window的概念，而不是任何时间都可以合入的。对于使用git作为版本工具的项目来说，这种方式其实更合适，因为拉分支合分支很方便，而且天然不会存在版本回退的问题。

## 参考资料
How the development process works
<https://www.kernel.org/doc/html/latest/process/2.Process.html#the-big-picture>

Everything you ever wanted to know about Linux -stable releases
<https://www.kernel.org/doc/html/latest/process/stable-kernel-rules.html>

Linux kernel version history
<https://en.wikipedia.org/wiki/Linux_kernel_version_history>

