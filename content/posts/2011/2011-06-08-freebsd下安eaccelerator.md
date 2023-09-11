---
title: FreeBSD下安eaccelerator
author: admin
type: post
date: 2011-06-08T12:33:47+00:00
url: /archives/9718
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - eAccelerator

---
> \# cd /usr/ports/www/eaccelerator
> #make install clean

安装完会提示在/usr/local/etc/php.ini文件末尾添加一行zend_extension=”/usr/local/lib/php/20090626/eaccelerator.so”,并创建临时目录/tmp/eaccelerator.

> #echo ‘zend_extension=”/usr/local/lib/php/20090626/eaccelerator.so”‘ >> /usr/local/etc/php.ini
> #mkdir /tmp/eaccelerator
> #chown www /tmp/ eaccelerator
> #chmod 0700 /tmp/eaccelerator