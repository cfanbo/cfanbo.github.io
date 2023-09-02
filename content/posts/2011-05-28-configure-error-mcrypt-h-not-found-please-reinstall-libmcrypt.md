---
title: 'configure: error: mcrypt.h not found. Please reinstall libmcrypt'
author: admin
type: post
date: 2011-05-28T13:02:54+00:00
url: /archives/9592
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - libmcrypt
 - php

---
今日参考以前的文章安装lnmp的时候,发现这次在安装php的时候竟然提示”configure: error: mcrypt.h not found. Please reinstall libmcrypt”,意思是，没有查找到mcrytp.h,需要安装libcrytp,以前安装了n次都没有问题的,在网上找了一个解决办法.

> wget [ftp://mcrypt.hellug.gr/pub/crypto/mcrypt/attic/libmcrypt/libmcrypt-2.5.7.tar.gz][1]
>
> tar -zxvf libmcrypt-2.5.7.tar.gz
> cd libmcrypt-2.5.7
> mkdir -p /usr/local/libmcrytp
> ./configure prefix=/usr/local/libmcrytp/
> make
> make install

然后再安装PHP



 [1]: ftp://mcrypt.hellug.gr/pub/crypto/mcrypt/attic/libmcrypt/libmcrypt-2.5.7.tar.gz