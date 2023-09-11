---
title: 同服务器多网站的安全设置（windows server+apache+php）
author: admin
type: post
date: 2010-04-19T01:45:04+00:00
url: /archives/3432
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - apache
 - php
 - 安全

---
在windows环境下，如果用IIS做webserver，可以配合ntfs为每个网站设置不同的用户权限，从而让一个网站的程序只能访问自己目 录下的内容.

而在windows的apache环 境下，由于apache默认是最高的system权限，因此非常危险，若不做安全设置，随便传一个php shell到任何一个网站上，就能控制整台服务器。

要实现这个目标，需要做以下设置：

1、在vhost中设置 open\_basedir，设置后，php程序将只能打开规定目录下的内容（此指令不受安全模式是否打开的限制）。如下。同时最好把php.ini的 upload\_tmp_dir 目录也添加进去，否则可能无法正常上传文件。

 ServerAdmin [admin@abc.com](mailto:admin@abc.com)

 DocumentRoot D:/abc

 ServerName [www.abc.com](http://www.abc.com/)

 ErrorLog logs/abc.com-error_log

 CustomLog logs/abc.com-access_log common

php_admin_value open_basedir “D:/abc;D:/php/temp”

 Options FollowSymLinks

 AllowOverride all

 Order allow,deny

 Allow from all

2、 在php.ini中打开安全模式，这样可以禁止exec,system等危险的函数被使用（ [查看被禁止的函数](http://www.google.cn/search?sourceid=navclient&hl=zh-CN&ie=UTF-8&rlz=1T4SUNA_zh-CN___CN207&q=php+%e5%ae%89%e5%85%a8%e6%a8%a1%e5%bc%8f+%e5%b1%8f%e8%94%bd%e7%9a%84%e5%87%bd%e6%95%b0)），因为使用这些函数可以查看整个服务器所有系统的内容,比如可以使用所有系统命令 dir,del,format等。

PHP 的安全模式是为了试图解决共享服务器（shared-server）安全问题而设立的。在结构上，试图在 PHP 层上解决这个问题是不合理的，但修改 WEB 服务器层和操作系统层显得非常不现实。因此许多人，特别是 ISP，目前使用安全模式。

3、在php.ini中禁用函 数：phpinfo()

来源： [http://www.phpchina.com/html/36/13636-8285.html](http://www.phpchina.com/html/36/13636-8285.html)