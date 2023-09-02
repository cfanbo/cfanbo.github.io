---
title: 'MySQL /bin/rm: cannot remove `libtoolT’: No such file or directory的解决办法'
author: admin
type: post
date: 2010-10-12T07:08:31+00:00
url: /archives/5986
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos

---
在 CentOS 5.5 下编译安装MySQL时出错：

/bin/rm: cannot remove \`[libtoolt][1]‘: No such file or directory

config.status: executing depfiles commands
config.status: executing libtool commands
/bin/rm: cannot remove \`libtoolT’: No such file or directory
config.status: executing default commands
configure: WARNING: unrecognized options: –with-low-mymory

Thank you for choosing MySQL!

Remember to check the platform specific part of the reference manual
for hints about installing MySQL on your platform.
Also have a look at the files in the Docs directory.

网上搜寻后，解决问题。具体方法是：

在执行./configure 之前，先执行：

> # autoreconf –force –install
> # libtoolize –automake –force
> # automake –force –add-missing
> \# ./configure –prefix=/usr/local/mysql/ –datadir=/var/lib/mysql

这次，不再出错了，问题解决。

顺便记录一下我的编译参数：

> ./configure –prefix=/usr/local/mysql/ –datadir=/var/lib/mysql –with-unix-socket-path=/tmp/mysql.sock –enable-assembler –with-extra-charsets=complex –enable-thread-safe-client –with-client-ldflags=all-static –with-mysqld-ldflags=all-static –with-big-tables –with-readline –with-ssl –with-embedded-server –enable-local-infile –with-plugins=innobase

 [1]: http://www.yanghengfei.com/tag/libtoolt/