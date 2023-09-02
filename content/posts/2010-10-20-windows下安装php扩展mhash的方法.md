---
title: windows下安装php扩展mhash的方法
author: admin
type: post
date: 2010-10-20T06:57:47+00:00
url: /archives/6248
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php

---

mhash是php hash加密算法很强大的扩展，但是php5.3.0以后就不包含这个程序包了，需要自己去下载: [http://sourceforge.net/projects/mhash](http://sourceforge.net/projects/mhash)。


php5.2.0默认不挂载，所以就需要自己手动去挂，windows版本安装比较容易.


首先.修改php.ini,启用扩展 **extension=php_mhash.dll**。


再次.因为php_mhash.dll需要libmhash.dll才能加载，所以把 **libmhash.dll** 复制到C:\WINDOWS\system32下去即可。


由此得到的启发就是，如果在windows下动态链接模块加载不了的，请将相关的libxxx.dll复制到C:\WINDOWS\system32下去即可。