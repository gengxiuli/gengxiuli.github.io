---
layout: post
title:  "Jekyll theme chirpy avatar update"
date:   2025-06-01
category:  blog
tags:   jekyll chirpy avatar
---

本网站基于 [jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy), 在 GitHub 上发布 post 的体验也不错，和写代码一样，git add, git commit, git push 三部曲。使用 github desktop + vscode 也很丝滑流畅。

但是本网站的用户体验还有一个痛点，就是所有页面左侧的图标一直是空白，因为不影响功能使用，所以也没有在意。今天在更新博客的时候，想起来研究一下，发现其实很简单，就是配置一下 _config.ym l文件中的 avatar 字段，指向项目中的 avatar 文件：

```yml
# the avatar on sidebar, support local or CORS resources
avatar: "assets/image/avatar.jpg"
```

其实在 chirpy 的 [wiki](https://github.com/cotes2020/jekyll-theme-chirpy/wiki) 页面作者也分享过 [Customize the Favicon](https://chirpy.cotes.page/posts/customize-the-favicon/) 的方法，这是网页中的 favicon 图标，avatar 是侧边栏的头像，修改方法是还是有区别的。

需要注意的是，从 [cotes2020/chirpy-starter](https://github.com/cotes2020/chirpy-starter) generated 的项目，aasets 中的 lib 目录是一个链接，其指向 [chirpy-static-assets](https://github.com/cotes2020/chirpy-static-assets)。个人的 avatar 是不能上传到这个目录的，因为你大概率没有这个权限，所以需要在 assets 新建一个目录存放个人文件。