---
title: FreeBSD系统下普通用户切换root用户,提示su:sorry的解决办法
author: admin
type: post
date: 2010-11-18T08:16:43+00:00
url: /archives/6706
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - su

---
FreeBSD系统下su:sorry的解决办法
在FreeBSD上要使用su命令成为root用户，不但要知道root的口令，还需要经过特别设置，否则就不能成功使用这个命令。这是因为 FreeBSD对执行su命令的用户进行了更严格的限制，能使用su命令的用户必须属于wheel组（root的基本属组，组ID为0），否则就不能通过 这个命令成为root用户。因此需要编辑组设置文件/etc/group，将需要超级用户权力的管理成员加入到wheel组中。
可以使用如下命令给普通用户su – root的权力：

> pw groupmod wheel -m
> pw user mod  -g wheel

或者直接修改/etc/group文件，把相应的用户加到wheell组就可以

> wheel:*:0:root,

FreeBSD系统下默认是不允许root用户直接通过ssh连接到服务器的，在安装FreeBSD系统时要创建一个额外的用户，切忌一定要把这个用户加入到wheel组中（如果不加入到这个组中的话就无法ssh），也可以安装完系统后创建用户，并把这个用户加入wheel组。