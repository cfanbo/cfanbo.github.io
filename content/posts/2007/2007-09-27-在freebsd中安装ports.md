---
title: 在FreeBsd中安装ports
author: admin
type: post
date: 2007-09-27T19:59:52+00:00
url: /archives/168
IM_contentdowned:
 - 1
categories:
 - 服务器

---
在[FreeBsd][1]中安装ports
一.首先进入要安装的port的目录
#cd /usr/ports/www/apache22
二.执行make命令进行编译
#make
会出现一些提示信息,一旦编译完,就会回到命令行,下一步是安装port,只要在make后面添加一个单词install即可.
三.安装port
#make install
会出现一些提示信息,完毕后会回到提示符,您就可以运行您安装的程序了
四.清除安装时产生的一些临时信息:
#make clean
清理工作目录是个好注意,这个目录中包含了全部在编译过程中用到的临时文件,这些文件不公会占用宝贵的磁盘空间,而且可能给升级port时带来麻烦.

至此,安装ports的步骤基本已经完成.

注:以上三个命令make,make install,make clean可以使用组合命令make install clean来代替.

 [1]: /?tag=freebsd