---
title: '在安装php的make时候,出现错误”chmod: cannot access `ext/phar/phar.phar’: No such file or directory”的解决办法'
author: admin
type: post
date: 2012-02-05T06:45:56+00:00
url: /archives/12482
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - php

---
在对php进行configure的时候,只需要在./configure的后面加上–without-pear 即可.