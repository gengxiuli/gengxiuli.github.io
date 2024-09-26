---
layout: post
title:  "多语言构建系统简介"
date:   2024-09-27
category: programming
tags:   build bazel buck
---

偶然从[byvoid的个人简历](https://byvoid.com/en/resume/)中知道了[buck](https://github.com/facebook/buck)。简单看了一下知道了这是一个facebook开源的编译工具，但是该github仓库已不再更新，而是转移到了[buck2](https://github.com/facebook/buck2)。buck2的介绍是fast multi-language build system，然后下面解释了buck2/buck构建系统的特点和由来。其中提到了多语言的支持，而这也是大型复杂系统的特点，编程语言多，依赖关系复杂，包含测试用例代码静态扫描等保证系统，传统的集成方式实现复杂且容易出错。buck2对编译过程进行了抽象，可以支持各种编程语言的构建，而且相对于buck构建速度更快更好用。