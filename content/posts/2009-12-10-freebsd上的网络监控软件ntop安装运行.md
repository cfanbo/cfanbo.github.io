---
title: FreeBSD上的网络监控软件ntop安装运行
author: admin
type: post
date: 2009-12-10T08:46:14+00:00
url: /archives/2702
IM_data:
 - 'a:1:{s:44:"/wp-content/uploads/2009/12/freebsd-ntop.gif";s:44:"/wp-content/uploads/2009/12/freebsd-ntop.gif";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ntop

---
![freebsd-ntop](/wp-content/uploads/2009/12/freebsd-ntop.gif)

FreeBSD上的网络监控软件ntop的安装步骤如下：

1、将相关的ports升级。（cvsup或其他方式如Sub….等）；

2、安装ntop

>

> # **cd /usr/ports/net/ntop**
>

>
>

> # **make install clean**

**#rehash**
>

3、安装完成后，可以手工启动 ntop。即输入ntop,运行即可。

如果是第一次启动ntop，系统会提示，输入admin的口令；

或在/etc/rc.conf文件中加入 **ntop_enable=”YES”** 自动启动ntop；

4 、设置密码

>

> /usr/local/bin/ntop -u nobody -A
>

输入查看密码

以进程服务形式启用ntop

>

> /usr/local/bin/ntop -u nobody -d
>

5、此时可以用netstat -an 或 **netstat -an |grep 3000** 查看3000的端口在监听；

6、在浏览器中，输入http://*.*.*.* :3000既可以进入相应的ntop网络管理监控界面。