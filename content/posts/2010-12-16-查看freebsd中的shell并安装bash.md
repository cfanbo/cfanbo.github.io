---
title: Freebsd下安装bash
author: admin
type: post
date: 2010-12-16T06:17:45+00:00
url: /archives/6921
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - bash
 - shell

---

FreeBSD下默认的shell为CSH，可以通过命令


> echo $SHELL

来查看系统默认的shell是哪一个的。


想知道FreeBSD都支持哪些shell,可以用下面的命令进行查看的


> #cat /etc/shells

默认只支持


> /bin/sh
>
> /bin/csh
>
> /bin/tcsh

这三种shell的,平时我们经常用bash 来写shell脚本,特别是对于那些从linux转过来的用户来说,bash可能说无所不在的.但freebsd默认情况下并不支持bash的,我们可以手动安装一下bash的,命令如下:

**1.安装bash**

> cd /usr/ports/shells/bash
>
> make install clean

**2. 在/bin目录下面做一个符号连接。**

> ln -s /usr/local/bin/bash /bin/bash

**3.加入bash**

> echo ‘/bin/bash’ >> /etc/shells

**4.更改用户shell**

> chsh -s /bin/bash root

**5.配置**

> vi ~/.profile
>
> alias ls=’ls -G’ #显示颜色
>
> alias ll=’ls -al’
>
> alias rm=’rm -i’ #确认删除
>
> alias mv=’mv -i’ #确认移动

**6.退出重新登录即生效**

以后就可以运行bash的shell脚本了.


如这里是一个重启nginx的命令shell


> #!/bin/bash
>
> kill -HUP `cat /var/run/nginx.pid`
>
> echo ‘nginx restart!’