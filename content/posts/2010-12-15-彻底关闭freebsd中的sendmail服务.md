---
title: 彻底关闭FreeBSD中的sendmail服务
author: admin
type: post
date: 2010-12-15T03:09:29+00:00
url: /archives/6894
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - sendmail

---
FreeBSD系统中的sendmail一直默认启动，而且不容易关闭。必须修改配置文件rc.conf，并一关闭几个相关进程才行。

在/etc/rc.conf文件中加入下面几行：

> sendmail_enable=”NO”
>
> sendmail\_submit\_enable=NO
>
> sendmail\_outbound\_enable=NO
>
> sendmail\_msp\_queue_enable=NO

重新启动系统。sendmail进程不再启动了。

试了一下，只要加一行，sendmail也不会启动了

在 /etc/rc.conf中加入

> sendmail_enable=”NONE”