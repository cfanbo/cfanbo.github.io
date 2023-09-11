---
title: 解决Default storage engine (InnoDB) is not available导致mysql无法启动的
author: admin
type: post
date: 2009-06-27T03:41:18+00:00
url: /archives/1929
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---
一次为了修改mysql的root用户密码，就启用了本机启动模式，可再次启用mysql时，却揭示：Default storage engine (InnoDB) is not available ，mysql无法启动，后搜索网络，得知应该是配置文件有错，这里提示：“060827  1:12:22 [ERROR] Default storage engine (InnoDB) is not available”
打开my.ini或my.cnf文件，找到default-storage-engine这一行，把它改成default-storage-engine=MyISAM。