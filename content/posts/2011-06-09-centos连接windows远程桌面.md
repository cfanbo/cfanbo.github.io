---
title: '[教程]centos连接windows远程桌面'
author: admin
type: post
date: 2011-06-09T08:23:09+00:00
url: /archives/9739
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos

---
大家都知道linux下都是用rdesktop来连接windows的远程桌面。注意只能在centos下的gui模式下运行，如果在cli下运行，则会提示以下错误：

> Autoselected keyboard map en-us

 ERROR: Failed to open display:

所以先安装rdesktop

 可以通过yum list看看有没有rdesktop包，可以看到有rdesktop.i386-1.4.1-4

 下面我们直接安装：

> shell> yum install rdesktop.i386

 －－－－过程省略—-

 安装完成后我们直接用

> shell> rdesktop -a 16 192.168.1.5:3389来连接windows远程桌面。 -a 16表示用16位颜色打开桌面，后面的ip地址是windows服务器地址 :3389是windows的远程桌面的端口号，其实默认的3389可以省略，如果调整了windows远程桌面的端口，这里就必须带上。

整个地球都知道rdesktop，有了它，我们可以从Solaris或者Linux使用Windows，当然Windows要开启Windows Terminal Service。虽然也有基于GTK+的tsclient做配置，我还是倾向直接使用命令行，不仅因为自己习惯使用console命令窗口，而且命令行可 以加入一些非常有用的选项。

**比如：**
./rdesktop -u adam -p adam -f -r clipboard:PRIMARYCLIPBOARD -r disk:sunray=/home/yz161846 oss-ww

 -u 和 -p: 指定用户名和密码

 -f : 默认全屏， 需要用Ctrl-Alt-Enter组合键进行全屏模式切换。

 -r clipboard:PRIMARYCLIPBOARD : 这个一定要加上，要不然不能在主机Solaris和服务器Windows直接复制粘贴文字了。贴中文也没有问题。

 -r disk:sunray=/home/yz16184 : 指定主机Solaris上的一个目录映射到远程Windows上的硬盘，传送文件就不用再靠Samba或者FTP了。

除了这些常用的选项，rdesktop也支持cdrom, floppy软盘的远程映射，详细可以参考rdesktop命令帮助。
./rdesktop -h

rdesktop程序可以自己从[www.rdesktop.org](http://www.rdesktop.org) 上获取代码编译，非常方便。也可以直接从www.sunfreeware.com下载一份可执行档。

当然如果你不满意这个开源软件，如果花钱选择像Windows的Remote Connection, 或者Citrix这种商业软件。