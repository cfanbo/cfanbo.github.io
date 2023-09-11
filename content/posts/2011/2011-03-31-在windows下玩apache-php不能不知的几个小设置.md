---
title: 在windows下玩apache-php不能不知的几个小设置
author: admin
type: post
date: 2011-03-31T12:50:09+00:00
url: /archives/8802
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - php

---
1、PHPIniDir “D:\PHP5″

这样不用每次都把php.ini拷贝到C:\Windows下

2、set Path=D:\PHP5;D:\PHP5\ext;%Path%

这样不用每次把那些dll拷贝到C:\Windows\system32下