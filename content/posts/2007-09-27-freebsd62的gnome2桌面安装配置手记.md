---
title: freebsd6.2的gnome2桌面安装配置手记
author: admin
type: post
date: 2007-09-27T23:14:44+00:00
url: /archives/169
IM_contentdowned:
 - 1
categories:
 - 服务器

---
1、安装[freebsd][1]6.2。
这里我选择的是最小化安装。

2、安装xorg。
pkg_add  -r xorg

3、安装gnome2。
pkg_add -r gnome2

4、生成、测试相关的配置文件

Xorg -configure
将生成xorg.conf.new文件在/root/目录下。
Xorg -configure /root/xorg.conf.new（6.2做这步时似乎必须加上/root/）
这里测试下生成的配置文件，会出现1个布满小格子的大方框，并且应该有一个鼠标箭头。
然后ctrl+alt+backspace返回文字符界面。
然后编辑一下xorg.conf.new文件，然后拷贝至/etc/X11/xorg.conf

5、配置窗口管理器
在/etc/rc.conf里加入gdm_enable=”YES”
然后重新启动，就可以自动进入GUI界面了。

 [1]: /?tag=freebsd