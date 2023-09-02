---
title: 在FreeBSD系统中用pkill命令踢出SSH在线登录用户
author: admin
type: post
date: 2012-01-11T12:55:57+00:00
url: /archives/12432
IM_contentdowned:
 - 1
categories:
 - 服务器

---
FreeBSD是一个多用户多任务的操作系统，用户可以在不同地方通过ssh连上FreeBSD服务器，在系统中我们可以使用w命令来查看当前在线登录用户。

> [root@host01 ~]# w
>
> 03:05:23 up 19 min, 3 users, load average: 0.00, 0.03, 0.05
> USER TTY FROM   LOGIN@ IDLE WHAT
> root **p0** 192.168.0.2 01:39 6:52 /usr/bin/perl
> root **p1** 192.168.0.31 01:45 0.00s w
> root **p2** 192.168.0.23 01:52 2.00s -bash

看到了吧，已经有3个用户登录到服务器了。接下来使用who am i 看那个是自己的登录终端，下面自己是pts/1

> [root@host01 ~]# who am i
> root**p1** 2009-08-02 03:06 (192.168.0.31)

接下来使用pkill命令将要其它的用户踢出,这里为p0和p2。


[root@host01 ~]# **pkill -kill -t p2**

再使用w命令查看，p2终端用户已经被踢掉了。

> [root@host01 ~]# w
>
> 03:07:28 up 21 min, 2 users, load average: 0.00, 0.02, 0.05
> USER TTY FROM   LOGIN@ IDLE WHAT
> root p0 192.168.0.2 01:39 6:52 /usr/bin/perl
> root p1 192.168.0.31 01:45 0.00s w

=========================================

比如某一用户通过SSH登陆进入系，会话为pts/0（设此用户为第一个通过SSH登陆进入系统的）。

> \# ps -aux | grep pts/0
> root    865  0.0  0.5  9400  4360  ??  Is    4:08AM   0:00.16 sshd: root@pts/0
> root    926  0.0  0.1  3492  1208  v1  S+    4:17AM   0:00.01 sshd: grep pts/0

开始踢他了：

> \# kill -9 865

或者

> #pkill -kill -t pts/0