---
title: 'configure: error: no acceptable C compiler found in $PATH的解决办法'
author: admin
type: post
date: 2010-09-15T11:07:14+00:00
url: /archives/5697
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux

---
原因：没有安装gcc编译环境

debian:/usr/local/httpd-2.2.9# apt-get install gcc 正在读取软件包列表… 完成

正在分析软件包的依赖关系树… 完成

将会安装下列额外的软件包：

gcc-4.1 libssp0

建议安装的软件包：

manpages-dev autoconf automake1.9 libtool flex bison gcc-doc gcc-4.1-doc

gcc-4.1-locales libc6-dev-amd64 lib64gcc1 lib64ssp0

推荐安装的软件包：

libc6-dev libc-dev libmudflap0-dev

下列【新】软件包将被安装：

gcc gcc-4.1 libssp0

共升级了 0 个软件包，新安装了 3 个软件包，要卸载 0 个软件包，有 0 个软件未被升 级。

需要下载 5052B/470kB 的软件包。

解压缩后会消耗掉 1401kB 的额外空间。

您希望继续执行吗？[Y/n]y

获取：1 http://debian.bestwiz.cn etch/main gcc 4:4.1.1-15 [5052B]

更换介质：请把标有

“Debian GNU/Linux 4.0 r0 \_Etch\_ – Official i386 NETINST Binary-1 20070407-11:29”

的碟片插入驱动器“/cdrom/”再按回车键

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝

又有新麻烦了，解决方法：

debian:/usr/local/httpd-2.2.9# vi /etc/apt/sources.list

注释取消:

#deb cdrom:[Debian GNU/Linux 4.0 r0 \_Etch\_ – Official i386 NETINST Binary-1 20070407-11:29]/ etch contrib main

deb http://debian.bestwiz.cn/debian/ etch main

\# Line commented out by installer because it failed to verify:

#deb-src http://debian.bestwiz.cn/debian/ etch main

deb http://security.debian.org/ etch/updates main contrib

deb-src http://security.debian.org/ etch/updates main contrib

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝

debian:/usr/local/httpd-2.2.9# aptitude install gcc

正在读取软件包列表… 完成

正在分析软件包的依赖关系树… 完成

正在读取扩展状态文件

正在初始化软件包状态… 完成

正在读取软件集说明档… 完成

创建标签数据库… 完成

下列软件包将被自动安装：

gcc-4.1 libc6-dev libmudflap0 libmudflap0-dev libssp0 linux-kernel-headers

下列“新”软件包将被安装。

gcc gcc-4.1 libc6-dev libmudflap0 libmudflap0-dev libssp0 linux-kernel-headers

0 个软件包被升级，新安装 7 个，0 个将被删除， 同时 0 个将不升级。

需要获取 465kB/5430kB 的存档。解包后将要使用 24.8MB。

您要继续吗？[Y/n/?]

正在编辑扩展状态信息… 完成

读取：1 http://debian.bestwiz.cn etch/main libssp0 4.1.1-21 [4492B]

读取：2 http://debian.bestwiz.cn etch/main gcc-4.1 4.1.1-21 [461kB]

已下载 465kB，耗时 0s (4795kB/s)

选中了曾被取消选择的软件包 libssp0。

(正在读取数据库 … 系统当前总共安装有 76283 个文件和目录。)

正在解压缩 libssp0 (从 …/libssp0\_4.1.1-21\_i386.deb) …

选中了曾被取消选择的软件包 gcc-4.1。

正在解压缩 gcc-4.1 (从 …/gcc-4.1\_4.1.1-21\_i386.deb) …

选中了曾被取消选择的软件包 gcc。

正在解压缩 gcc (从 …/gcc\_4%3a4.1.1-15\_i386.deb) …

选中了曾被取消选择的软件包 linux-kernel-headers。

正在解压缩 linux-kernel-headers (从 …/linux-kernel-headers\_2.6.18-7\_i386.deb) …

选中了曾被取消选择的软件包 libc6-dev。

正在解压缩 libc6-dev (从 …/libc6-dev\_2.3.6.ds1-13etch5\_i386.deb) …

选中了曾被取消选择的软件包 libmudflap0。

正在解压缩 libmudflap0 (从 …/libmudflap0\_4.1.1-21\_i386.deb) …

选中了曾被取消选择的软件包 libmudflap0-dev。

正在解压缩 libmudflap0-dev (从 …/libmudflap0-dev\_4.1.1-21\_i386.deb) …

正在设置 libssp0 (4.1.1-21) …

正在设置 gcc-4.1 (4.1.1-21) …

正在设置 gcc (4.1.1-15) …

正在设置 linux-kernel-headers (2.6.18-7) …

正在设置 libc6-dev (2.3.6.ds1-13etch5) …

正在设置 libmudflap0 (4.1.1-21) …

正在设置 libmudflap0-dev (4.1.1-21) …

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝

安装完成

查看gcc版本：

debian:/home/liufei# gcc –version

gcc (GCC) 4.1.2 20061115 (prerelease) (Debian 4.1.1-21)

Copyright (C) 2006 Free Software Foundation, Inc.

This is free software; see the source for copying conditions. There is NO

warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

debian:/home/liufei# gcc -v

Using built-in specs.

Target: i486-linux-gnu

Configured with: ../src/configure -v –enable-languages=c,c++,fortran,objc,obj-c++,treelang –prefix=/usr –enable-shared –with-system-zlib –libexecdir=/usr/lib –without-included-gettext –enable-threads=posix –enable-nls –program-suffix=-4.1 –enable-_\_cxa\_atexit –enable-clocale=gnu –enable-libstdcxx-debug –enable-mpfr –with-tune=i686 –enable-checking=release i486-linux-gnu

Thread model: posix

gcc version 4.1.2 20061115 (prerelease) (Debian 4.1.1-21)

debian:/home/liufei# dpkg -l gcc

期望状态=未知(u)/安装(i)/删除(r)/清除(p)/保持(h)

| 当前状态=未(n)/已安装(i)/仅存配置(c)/仅解压缩(U)/配置失败(F)/不完全安装(H)

|/ 错误？=(无)/保持(?)/须重装(R)/两者兼有(#) (状态，错误：大写=故障)

||/ 名称         版本         简介

+++-==============-==============-============================================ii gcc            4.1.1-15       The GNU C compiler