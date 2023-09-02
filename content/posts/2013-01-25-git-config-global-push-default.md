---
title: git config –global push.default
author: admin
type: post
date: 2013-01-25T08:41:38+00:00
url: /archives/13611
categories:
 - 服务器
tags:
 - git

---
参考教程 [http://blog.haohtml.com/archives/10093](http://blog.haohtml.com/archives/10093) 刚安装的git最新版本，发现有些命令发生了一些变化.

> [web@bogon www]$ git push
> warning: push.default is unset; its implicit value is changing in
> Git 2.0 from ‘matching’ to ‘simple’. To squelch this message
> and maintain the current behavior after the default changes, use:
>
> git config –global push.default matching
>
> To squelch this message and adopt the new behavior now, use:
>
> git config –global push.default simple
>
> See ‘git help config’ and search for ‘push.default’ for further information.
> (the ‘simple’ mode was introduced in Git 1.7.11. Use the similar mode
> ‘current’ instead of ‘simple’ if you sometimes use older versions of Git)
>
> Counting objects: 4, done.
> Compressing objects: 100% (2/2), done.
> Writing objects: 100% (3/3), 263 bytes, done.
> Total 3 (delta 1), reused 0 (delta 0)

再次执行上面的命令的时候，可能会提示以下信息：

> fatal: The current branch master has no upstream branch.
> To push the current branch and set the remote as upstream, use
>
> git push –set-upstream origin master

这个时候只需要执行一下命令

> git push –set-upstream origin master

即可