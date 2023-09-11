---
title: freebsd 7.0 vsftpd如何启动!!
author: admin
type: post
date: 2008-10-29T12:24:10+00:00
url: /archives/486
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - vsftpd

---
在etc/rc.conf中添加
vsftpd_enable=”YES”

/usr/local/etc/vsftpd.conf中添加

> listen=YES
> background=YES

就可以了,还真是挺复杂,每个软件安装了都要修改配置文件才能启动!!
如果出现错误

**500 OOPS: vsftpd: cannot locate user specified in ‘ftp_username’:ftp**在vsftpd.conf中加入了ftp_username=xxx（用户）

**以下命令可以用来重启vsftpd服务**
\# /usr/local/etc/rc.d/vsftpd restart