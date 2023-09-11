---
title: 解决vsftpd虚拟用户没有chmod权限的问题
author: admin
type: post
date: 2011-02-26T11:25:54+00:00
url: /archives/7852
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - vsftpd

---
参考(已经修正)，在下面搞了个ftp,结果发现vsftpd的虚拟用户无法获得chmod权限,后来找了找，解决办法如下：

修改配置文件

#让虚用户获得本地用户权限
virtual_use_local_privs=YES
#开启chmod命令
chmod_enable=YES