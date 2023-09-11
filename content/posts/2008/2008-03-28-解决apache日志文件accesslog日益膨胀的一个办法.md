---
title: '解决Apache日志文件ACCESS.LOG日益膨胀的一个办法:'
author: admin
type: post
date: 2008-03-28T23:43:10+00:00
url: /archives/281
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache

---
将httpd.conf中CustomLog logs/access.log common 改成

CustomLog “|c:/apache/bin/rotatelogs c:/apache/logs/%Y\_%m\_%d.access.log 86400 480” common

重启Apache

其中c:/apache/是你安装apache的路径
这样每一天生成一个日志文件