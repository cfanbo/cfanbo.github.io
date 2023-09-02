---
title: FreeBSD上安装sudo
author: admin
type: post
date: 2010-11-03T13:46:47+00:00
url: /archives/6548
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - sudo

---
下面来安装并使用sudo吧！

1）安装sudo,从 ports 安装

> #cd /usr/ports/security/sudo
> #make install clean

2)配置sudo
sudo的配置文件在/usr/local/etc/sudoers里面。sudo的配置文件不应直接编辑，而应使用

> #visudo

注意，不是vi sudo，而是visudo一个命令的。
下面是配置sudo的简单方法：

找到

> root           ALL=(ALL) ALL

这一行，在其下面加入：

> 用户名 ALL=(ALL)ALL

:wq退出即可。
导读： [su和sudo命令的区别与使用技巧](http://blog.haohtml.com/index.php/archives/6546)