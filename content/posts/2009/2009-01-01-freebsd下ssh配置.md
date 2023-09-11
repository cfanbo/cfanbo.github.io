---
title: FreeBSD下SSH配置
author: admin
type: post
date: 2009-01-01T11:02:51+00:00
excerpt: |
 sshd的配置文件一般位于/etc/ssh/sshd_config。

 　　终端下:#ee /etc/ssh/sshd_config

 　　---------------------------------------------

 　　#Protocol 2,1

 　　修改为：

 　　Protocol 2

 　　#ListenAddress 0.0.0.0

 　　修改为：

 　　ListenAddress 0.0.0.0

 　　#PermitRootLogin yes

 　　修改为

 　　PermitRootLogin yes

 　　（Linux上默认允许root用户登录，此处可不修改。）
url: /archives/778
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ssh

---
sshd的配置文件一般位于/etc/ssh/sshd_config。

终端下:#ee /etc/ssh/sshd_config

———————————————

#Protocol 2,1

修改为：

Protocol 2

#ListenAddress 0.0.0.0

修改为：

ListenAddress 0.0.0.0

#PermitRootLogin yes

修改为

PermitRootLogin yes

另把

#PasswordAuthenticationno

PasswordAuthentication yes

即可．

（Linux上默认允许root用户登录，此处可不修改。）

编辑**/etc/rc.conf**
最后加入:**sshd_enable=”yes”**

修改完成后重启sshd:

/etc/rc.d/sshd restart

——————————-

现在即可ssh登陆.