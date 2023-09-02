---
title: Call to undefined function curl_init()错误解决
author: admin
type: post
date: 2011-08-30T09:26:46+00:00
url: /archives/11060
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php

---
提示不支持这个函数，于是在php.ini文件里启用了扩展，把前面的;去掉，重启apache，竟然不起作用．后来查找了一下，原来还需要两个dll(libeay32.dll、ssleay32.dll)文件支持，将dll复制到c:/windows/system32目录里．然后重启apache即可．

特在此记录一下．