---
layout: post
title:  "多语言构建系统简介"
date:   2024-09-27
category: programming
tags:   build bazel buck blade cmake
---

偶然从[byvoid的个人简历](https://byvoid.com/en/resume/)中知道了[buck](https://github.com/facebook/buck)。简单看了一下知道了这是一个facebook开源的编译工具，但是该github仓库已不再更新，而是转移到了[buck2](https://github.com/facebook/buck2)。buck2的介绍是fast multi-language build system，然后下面解释了buck2/buck构建系统的特点和由来。其中提到了多语言的支持，而这也是大型复杂系统的特点，编程语言多，依赖关系复杂，包含测试用例代码静态扫描等保证系统，传统的集成方式实现复杂且容易出错。buck2对编译过程进行了抽象，可以支持各种编程语言的构建，而且相对于buck构建速度更快更好用。

buck2官网有个[Why Buck2](https://buck2.build/docs/about/why/)的页面，里面回答了三个问题:[why does Buck2 exist](https://buck2.build/docs/about/why/#why-does-buck2-exist)，[what's different about Buck2](https://buck2.build/docs/about/why/#whats-different-about-buck2)和[why use Buck2](https://buck2.build/docs/about/why/#why-use-buck2)，需要注意的是，虽然这里的很多比较基准都是buck，但页面内同时也标注了，这些区别同样适用于其他构建系统如bazel。

下面我们再来看看google开源的构建工具bazel,github仓库在[这里](https://github.com/bazelbuild/bazel)，但是官网主页[https://bazel.build](https://bazel.build)在手机浏览器上却一直无法访问(2024年9月27日验证)，目前可以移步参考这里[https://bazel.google.cn/](https://bazel.google.cn/)。在介绍页面bazel也提到了多语言多平台的支持:
> One tool, multiple languages: Build and test Java, C++, Android, iOS, Go, and a wide variety of other language platforms. Bazel runs on Windows, macOS, and Linux.
具体其他使用方法，可以参考官网手册，也可以在具体实践中学习摸索这里就不做过多介绍了。

除了上面提到的多语言构建系统外，还有很多针对特定语言的构建系统，比如[CMake](https://cmake.org/)就针对的是C++/C，最近在Windows平台下构建Wireshark就用到了CMake。CMake在这里可以根据环境配置生成Visual Studio构建用的sln文件，也可以生成Ninja构建使用的ninja文件，当然还可以生成传统Make使用的Makefile文件。CMake还支持代码外编译(Out-of-source builds)，即构建输出文件和源代码文件完全分开，便于代码管理和部署。国内的腾讯也开源了构建系统[blade](https://github.com/chen3feng/blade-build),功能和上面的bazel类似，这里有篇[使用指南](https://zzy979.github.io/posts/blade-build-tool/)，其中还提到构建系统和编译系统的区别:
> 构建(build)和编译(compile)不同——编译器负责将源代码转换为库文件或可执行文件；构建工具负责分析构建目标之间的依赖关系，并调用编译器来生成构建目标。

其实构建系统和编译系统，版本控制工具都可以集成起来，实现更加灵活和强大的功能，但是这需要很强的研发能力。[谷歌对微软：代码管理工具哪家强？](https://mp.weixin.qq.com/s/ckrH72rBp7_GT1UlfQsUaw)其中提到谷歌的一些内部做法，值得其他有需求的公司或者项目借鉴，另外其中还提到bazel的由来。