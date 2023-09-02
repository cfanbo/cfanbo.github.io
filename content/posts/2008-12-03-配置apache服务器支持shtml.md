---
title: 配置apache服务器支持shtml
author: admin
type: post
date: 2008-12-03T03:39:57+00:00
excerpt: |
 服务器采用shtml速度会比html慢，比php快。

 shtml的特点就是能够进行页面包含，能够局部更新页面包含部分。广泛采用可以很容易解决网页中的广告问题，不需要更新全面静态页面。而只需更新一个包含页面即可。

 apache下配置服务器支持shtml

 打开文件：httpd.conf

 去掉前面的 #LoadModule include_module modules/mod_include.so


 Options Indexes FollowSymLinks Includes
 AllowOverride Options FileInfo
 Order allow,deny
 Allow from all


url: /archives/665
IM_contentdowned:
 - 1
categories:
 - 程序开发
 - 服务器
tags:
 - apache
 - shtml
 - ssi

---
服务器采用shtml速度会比html慢，比php快。

shtml的特点就是能够进行页面包含，能够局部更新页面包含部分。广泛采用可以很容易解决网页中的广告问题，不需要更新全面静态页面。而只需更新一个包含页面即可。

apache下配置服务器支持shtml

打开文件：httpd.conf

去掉前面的 #LoadModule include\_module modules/mod\_include.so


Options Indexes FollowSymLinks Includes

AllowOverride Options FileInfo
Order allow,deny
Allow from all

找到下面两句，去掉前面的#

AddType text/html .shtml
AddOutputFilter INCLUDES .shtml

重启apache即可。

建立页面：

测试

file为相对于当前文档的路径。

virtual为相对于虚拟目录的路径。
如果需要让所有的html文件支持shtml.只需要修改上面一句。

AddOutputFilter INCLUDES .html