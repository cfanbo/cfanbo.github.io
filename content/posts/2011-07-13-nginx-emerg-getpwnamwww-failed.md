---
title: 'nginx [emerg]: getpwnam(“www”) failed'
author: admin
type: post
date: 2011-07-13T02:56:28+00:00
url: /archives/10406
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx

---
在配置nginx提示如下错误时：

> [emerg]: getpwnam(“www”) failed

**解决方案：**
在nginx.conf中 把#user nobdy改为user www www既可.

如果还提示同样的错误，请检查www组和www用户是否存在，不存在的话，直接创建即可

> /usr/sbin/groupadd www
> /usr/sbin/useradd -g www www