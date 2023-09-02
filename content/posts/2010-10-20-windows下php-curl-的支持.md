---
title: windows下php curl 的支持
author: admin
type: post
date: 2010-10-20T07:19:16+00:00
url: /archives/6254
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php

---
windows下的php配置，只是增加curl的扩展。
1、拷贝PHP目录中的libeay32.dll 和 ssleay32.dll 两个文件到 c:\windows\system32 目录。
2、修改php.ini。去掉 extension = php_curl.dll 前面的分号。
重启WEB服务即可！