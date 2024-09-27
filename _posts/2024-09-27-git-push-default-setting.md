---
layout: post
title:  git push.default setting
date:   2024-09-27
category: programming
tags:   git
---

在执行git push的时候打印了下面一段话：

> warning: push.default is unset; its implicit value is changing in
Git 2.0 from 'matching' to 'simple'. To squelch this message
and maintain the current behavior after the default changes, use:

>  git config --global push.default matching

> To squelch this message and adopt the new behavior now, use:

>  git config --global push.default simple

> See 'git help config' and search for 'push.default' for further information.
(the 'simple' mode was introduced in Git 1.7.11. Use the similar mode
'current' instead of 'simple' if you sometimes use older versions of Git)

大意是说当前Git(version 1.8.3.1)push.default未设置，而从Git 2.0开始，其默认设置由"matching"修改为"simple"。所以你需要显示的设置其默认值来关闭上述提示信息。

根据上述提示信息，使用git help config查看了帮助信息，其中push.default对所有支持的模式进行了详细的介绍，具体如下(git version 2.34.1下执行git help config)：

       push.default
           Defines the action git push should take if no refspec is given (whether from the command-line, config, or elsewhere). Different values are well-suited for specific
           workflows; for instance, in a purely central workflow (i.e. the fetch source is equal to the push destination), upstream is probably what you want. Possible values
           are:

           •   nothing - do not push anything (error out) unless a refspec is given. This is primarily meant for people who want to avoid mistakes by always being explicit.

           •   current - push the current branch to update a branch with the same name on the receiving end. Works in both central and non-central workflows.

           •   upstream - push the current branch back to the branch whose changes are usually integrated into the current branch (which is called @{upstream}). This mode only
               makes sense if you are pushing to the same repository you would normally pull from (i.e. central workflow).

           •   tracking - This is a deprecated synonym for upstream.

           •   simple - pushes the current branch with the same name on the remote.

               If you are working on a centralized workflow (pushing to the same repository you pull from, which is typically origin), then you need to configure an upstream
               branch with the same name.

               This mode is the default since Git 2.0, and is the safest option suited for beginners.

           •   matching - push all branches having the same name on both ends. This makes the repository you are pushing to remember the set of branches that will be pushed out
               (e.g. if you always push maint and master there and no other branches, the repository you push to will have these two branches, and your local maint and master
               will be pushed there).

               To use this mode effectively, you have to make sure all the branches you would push out are ready to be pushed out before running git push, as the whole point of
               this mode is to allow you to push all of the branches in one go. If you usually finish work on only one branch and push out the result, while other branches are
               unfinished, this mode is not for you. Also this mode is not suitable for pushing into a shared central repository, as other people may add new branches there, or
               update the tip of existing branches outside your control.

               This used to be the default, but not since Git 2.0 (simple is the new default).

综上所述，设置成Simple对大多数使用场景够用了。于是执行git config --global push.default simple，之后上述提示信息消失。