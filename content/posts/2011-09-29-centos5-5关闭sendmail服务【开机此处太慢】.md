---
title: CentOS5.5关闭sendmail服务【开机此处太慢】
author: admin
type: post
date: 2011-09-29T03:54:43+00:00
url: /archives/11573
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - sendmail

---
sendmail服务在系统启用的时候特别的慢，平时用的也不多的，所以为了安全直接将此服务关闭．并加速机器启用速度．

**1，关闭sendmail服务**

/etc/rc.d/init.d/sendmail stop

Shutting down sendmail:　　　　　　　　 [ OK ]
Shutting down sm-client: 　　　　　　　　[ OK ]

**2，关闭sendmail自启动**

[root@lsp ~]# chkconfig sendmail off



**3，确认sendmail自启动已被关闭（都为off就OK）**

[root@lsp ~]# chkconfig –list sendmail

sendmail 0:off 1:off 2:off 3:off 4:off 5:off 6:off

————–

chkconfig –list 可以用来查看所有的服务

如果提示chkconfig命令找不到，可使用/sbin/chkconfig的形式