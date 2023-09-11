---
title: Error:/etc/fstab:Read-only file system错误的解决办法
author: admin
type: post
date: 2010-12-26T09:23:19+00:00
url: /archives/7193
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - fstab

---
在单用户模式下，修改/etc/fstab的时候出现这个错误：

> Error:/etc/fstab:Read-only file system

Read-only file system 的原因很多。先重启一下的，看看能否解决的,如果重启还是解决不了，用命令：

修改挂载点/的权限为可读取模式：

> mount -o  rw /dev/ad0s1a  /

此时可用mount查看

> #mount
> /dev/ad0s1a on / (ufs, local)
> devfs on /dev(devfs, local, multilabel)

然后再编辑/etc/fstab，保存即可.

或者用这个命令：

或者用

> #fsck -fy /
> #mount -u /

自己没有测试过，网上找的。

相关文档：