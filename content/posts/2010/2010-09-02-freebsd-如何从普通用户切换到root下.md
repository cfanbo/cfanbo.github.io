---
title: FreeBSD 如何从普通用户切换到root下
author: admin
type: post
date: 2010-09-02T03:23:49+00:00
url: /archives/5424
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - su

---
OS:freebsd 7.2

在FreeBSD 7.2下，通过ssh客户端连接到FreeBSD端，用普通的用户登录，执行下列命令报错：

$ su –
su: Sorry
$ su
su: Sorry

原因:在FreeBSD上要使用 su命令成为root用户，不但要知道root的口令，还需要经过特别设置，否则就不能成功使用这个命令。这是因为 FreeBSD对执行su命令的用户进行了更严格的限制，能使用su命令的用户必须属于wheel组（root的基本属组，组ID为0），否则就不能通过这个命令成为root用户。

因此需要编辑组设置文件/etc/group，将需要超级用户权力的管理成员加入到wheel组中。

用 root用户登录，修改/etc/group文件，在wheel组中添加普通用户，操作如下：

#ee /etc/group

#wheel:*:0:root,pup (在wheel组中，增加pup用户)

按ESC,选择a)leave editor—->选择a)save changes,保存退出！

使用pup用户ssh登录

$ su –
Password:
#

成功切换到root权限！

参考: