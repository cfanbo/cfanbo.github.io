---
title: git remote用法总结
author: admin
type: post
date: 2013-01-25T08:24:20+00:00
url: /archives/13607
categories:
 - 其它
tags:
 - git

---
**git remote**

git remote显示所有的remote(加-v显示详细信息)。
git remote add \[shortname\] \[url\]用来添加remote。
git fetch [remote-name]只会pull下来全部的更动，但不会自动merge，但是git pull会自动merge。
git remote show [remote-name]可以看到一个remote的详细信息。
git remote rename old new 用来改变一个remote的名字。
git remote rm [remote-name]删除一个remote。
git remote 不带参数，列出已经存在的远程分支，例如：
#git remote
origin_apps

git remote -v | –verbose 列出详细信息，在每一个名字后面列出其远程url，例如：
#git remote -v
origin_apps     gitolite@scm:apps/Welcome.git (fetch)
origin_apps     gitolite@scm:apps/Welcome.git (push)
需要注意的是，如果有子命令，-v | –verbose需要放在git remote与子命令中间。

git remote add name url 在url创建名字为name的仓库（Adds a remote named  for the repository at ）
name为远程仓库的名字

git remote show name 必须要带name，否则git remote show的作用就是git remote，给出remote name的信息。