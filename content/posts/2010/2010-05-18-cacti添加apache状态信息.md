---
title: cacti添加apache状态信息
author: admin
type: post
date: 2010-05-18T01:45:34+00:00
url: /archives/3641
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - CACTI

---
对Apache Server Status的启用状态信息
对Apache的状态管理的模块是LoadModule status_module modules/mod_status.so,所以需要在配置文件httpd.conf里启用这个模块,所前面的#去掉.然后将”#Include conf/extra/httpd-info.conf“前面的#也去掉,打开**extra/httpd-info.conf**文件,启用

> ExtendedStatus On

配置Apache Server Status的权限

>
> SetHandler server-status
> Order Deny,Allow
> Deny from all
> Allow from 10.0.10.22
>

下载CACTI模板和脚本

[http://forums.cacti.net/about25227.html&highlight=apachestats](http://forums.cacti.net/about25227.html&highlight=apachestats)

在上面的地址下载一个叫 **ApacheStats08.zip** 的,中间有二个文件,一个处理脚本php的,另一个是xml的文件.

1.其中的ss_apache_stats.php是脚本文件,它是一个php的文件,放到你的 **cacti/scripts/** 下面.

2.接下来在cacti界面导入 **cacti_host_template_webserver_-_apache.xml** 这个文件

3.你就可以在cacti中加入这些设置.就不细写了,如下

被监测的apache服务器需要向上面一样,打开mod_status功能，记的设置好权限访问



相关文章:

[http://blog.haohtml.com/archives/3635](http://blog.haohtml.com/archives/3635)