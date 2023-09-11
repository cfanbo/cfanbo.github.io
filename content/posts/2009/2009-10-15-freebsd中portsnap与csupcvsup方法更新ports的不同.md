---
title: FreeBSD中portsnap与csup,cvsup方法更新ports的不同
author: admin
type: post
date: 2009-10-15T04:49:20+00:00
excerpt: |
 从6.0开始，freebsd升级ports就不再需要cvsup了，而是用portsnap，

 一、portsnap与cvsup的区别在于：

 1、portsnap有数字签名，较安全，cvsup没有。

 2、portsnap是打包压缩下载，所以会比cvsup快一些，当然除了第一次使用。

 二、使用方法是：

 第一次使用：portsnap fetch extract

 以后再用：portsnap fetch update

 还可以放在cron里定时升级：portsnap cron update

 需要注意的是不要portsnap和cvsup混合使用。
url: /archives/2520
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - portsnap

---

从6.0开始，freebsd升级ports就不再需要cvsup了，而是用portsnap，

一、portsnap与cvsup的区别在于

：

1、portsnap有数字签名，较安全，cvsup没有。

2、portsnap是打包压缩下载，所以会比cvsup快一些，当然除了第一次使用。

二、使用方法是：

第一次使用：portsnap fetch extract

以后再用：portsnap fetch update

还可以放在cron里定时升级：portsnap cron update

需要注意的是不要portsnap和cvsup混合使用。

第一次使用输入portsnap fetch extract回车即可，因为有几十兆的文件需要下载，需要等待一段时间。

如果用户没有安装ports，这个命令是无效的，需要通过sysinstall来安装ports

修改更新服务器地址的方法:

/etc/portsnap.conf 里面更改

SERVERNAME=portsnap.hshh.org

提供几个postsnap更新的服务器地址

portsnap.hshh.org

portsnap2.hshh.org

portsnap3.hshh.org (网通)

portsnap4.hshh.org

从6.0开始，freebsd升级ports就不再需要cvsup了，而是用portsnap，

一、portsnap与cvsup的区别在于：

1、portsnap有数字签名，较安全，cvsup没有。

2、portsnap是打包压缩下载，所以会比cvsup快一些，当然除了第一次使用。

二、使用方法是：

第一次使用：portsnap fetch extract

以后再用：portsnap fetch update

还可以放在cron里定时升级：portsnap cron update

需要注意的是不要portsnap和cvsup混合使用。

第一次使用输入portsnap fetch extract回车即可，因为有几十兆的文件需要下载，需要等待一段时间。

如果用户没有安装ports，这个命令是无效的，需要通过sysinstall来安装ports

修改更新服务器地址的方法:

/etc/portsnap.conf 里面更改

SERVERNAME=portsnap.hshh.org

提供几个postsnap更新的服务器地址:

portsnap.hshh.org

portsnap2.hshh.org

portsnap3.hshh.org (网通)

portsnap4.hshh.org