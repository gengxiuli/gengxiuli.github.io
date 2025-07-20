---
layout: post
title:  "jekyll theme chirpy comments support"
date:   2025-07-20
category:  blog
tags:   jekyll chirpy comments
---

本网站基于 [jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy), 之前的[文章](https://gengxiuli.com/posts/jekyll-theme-chirpy-avatar-update/)介绍过如何更新avatar图标，这里来说说如何支持comments，也就是增加评论功能。

[https://chirpy.cotes.page/](https://chirpy.cotes.page/)的comments使用的是[giscus](https://giscus.app/zh-CN),看起来也不错，需要github登录才可以评论，可以在一定程度上屏蔽垃圾评论。

步骤1：

在项目的_config.yml中选择支持的评论系统，chirpy支持disqus，utterances 和 giscus，这里我们选择giscus。

```yml
comments:
  # Global switch for the post-comment system. Keeping it empty means disabled.
  provider: giscus # [disqus | utterances | giscus]
  # The provider options are as follows:
```

步骤2：

在giscus网站设置自己需要的配置，包括语言，所在的仓库(需要一些设置，参考步骤3)，页面与discussion 映射关系，Discussion 分类，特性和主题。其中除了仓库名称必须填写之外，其他的设置可以使用默认的。 github仓库名称是：用户名/仓库名

步骤3：

1) 设置github对应的仓库为public属性，具体参考[这里](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/setting-repository-visibility#making-a-repository-public)。
2) 在github对应的仓库安装giscus,具体参考[这里](https://github.com/apps/giscus)。
3) 在github对应的仓库启用discussions功能,具体参考[这里](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/enabling-or-disabling-github-discussions-for-a-repository)。

上面这3个设置的链接在giscus的网站[https://giscus.app/zh-CN](https://giscus.app/zh-CN)也可以找到，放在这里以便直接跳转参考。

步骤4：

这一步很关键，在[https://giscus.app/zh-CN](https://giscus.app/zh-CN)的"启用giscus"部分中，选择下面"="号之后的内容，复制到chirpy中的_config.yml的comments中giscus部分。

From : 

```javascript
    data-repo="[在此输入仓库]"
    data-repo-id="[在此输入仓库 ID]"
    data-category="[在此输入分类名]"
    data-category-id="[在此输入分类 ID]"
```

To :

```yml
  giscus:
    repo: # <gh-username>/<repo>
    repo_id:
    category:
    category_id:
```

如果没有设置这一步，虽然看起来giscus的评论系统启用了，但是当你登录评论提交的时候，会提示你找不到discussions。

步骤5：

保存_config.yml文件，然后等待github的action自动构建和部署pages，最终你的评论就可以使用了。

参考链接：
https://ittousei.github.io/posts/customize-my-blog/
