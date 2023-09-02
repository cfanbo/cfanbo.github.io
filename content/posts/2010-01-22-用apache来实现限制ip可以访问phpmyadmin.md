---
title: 用apache来实现限制ip可以访问phpmyadmin
author: admin
type: post
date: 2010-01-22T06:30:47+00:00
url: /archives/2860
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - apache
 - phpmyadmin

---
为了安全,只允许固定ip才可以访问phpmyadmin,这个由于没有找到在phpmyadmin配置的地方,所以这里用apache来实现这个功能


ServerAdmin www@163.com
DocumentRoot “D:\site\phpMyAdmin”
ServerName php.haohtml.com
DirectoryIndex index.php index.shtml


Options Indexes MultiViews
AllowOverride None
order deny,allow
Allow from 192.168.0.7
Deny from all
#    Allow from all
Options FollowSymLinks Includes

经过上面一系列的配置,将只允许ip为192.168.0.7的ip才可以访问phpmyadmin,其它的ip都不允许访问.