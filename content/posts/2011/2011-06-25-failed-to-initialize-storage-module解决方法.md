---
title: Failed to initialize storage module解决方法
author: admin
type: post
date: 2011-06-25T11:32:20+00:00
url: /archives/9999
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php

---
今天更新了一下自己的cms，然后后台就提示登陆不了，报错如下：Failed to initialize storage module。

**解决方法有两种如下：**

1。在报错的文件里的session start();之前加入如下代码：ini\_set(‘session.save\_handler’, ‘files’); 。这种方法适合租用空间的用户使用。

2。在php.ini文件里，显式指定session的save_path(比如 c:/temp)然后重启web服务。如果服务器的管理权限属于你，那还是这样改比较方便。

原因分析：php5一个安全模式的bug，默认session的save_path是系统的临时目录，这样会要校验权限。

PHP中使用SESSION后出现Failed to initialize storage module错误的解决方法：
在session start之前加入以下这句话
ini_set(‘session.save_handler’, ‘files’);