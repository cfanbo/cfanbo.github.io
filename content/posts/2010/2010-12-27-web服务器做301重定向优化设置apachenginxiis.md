---
title: web服务器做301重定向优化设置(apache,nginx,iis)
author: admin
type: post
date: 2010-12-27T11:20:07+00:00
url: /archives/7306
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - iis
 - nginx

---
做网站优化的时候，网站301重定向是一个非常重要的操作方式。这样能够把多个域名的权重集中到一个域名，例如：www.haohtml.com和 haohtml.com，我们把haohtml.com重定向到www.haohtml.com，搜索引擎在搜索的时候，会把搜索结果或者Google评级的时候都集 中到www.haohtml.com。但是，在设置301的时候，会根据服务器的不同，有不同的设置。

一般情况下，网站301重定向可以分为IIS、Apache、Nginx三种，接下来我说明一下在虚拟主机下如何实现301重定向。

IIS:如果使用ASP的网站程序，可以使用asp脚本实现301重定向：写入header.asp或者其他头部文件。
这种方法最为简单，当然空间支持ISAPI 可以在网站根目录新建一个httpd.ini
将haohtml.com转移到www.haohtml.com上

> [ISAPI_Rewrite]
> \# 3600 = 1 hour
> CacheClockRate 3600
> RepeatLimit 32
> RewriteCond Host: ^haohtml.com.com$
> RewriteRule (.*) http://www.haohtml.com$1 [I,RP]

Apache:当服务器是apache的时候，只要在网站根目录新建一个.htaccess，写入以下代码：

> rewriteEngine on
> rewriteCond %{http_host} ^haohtml.com [NC]
> rewriteRule ^(.*)$ http://www.haohtml.com/$1 [R=301,L]

Nginx:如果web服务器是Nginx，需要修改绑定的域名的配置文件，例如：haohtml.com.conf

在行 ：server_name www.haohtml.com haohtml.com;下面添加

> if ($host != ‘www.haohtml.com’ ) {
> rewrite ^/(.*)$ http://www.haohtml.com/$1 permanent;
> }

实现重定向的方法还有很多，这里仅列举最常见的。如果你已经设置好301重定向，请务必使用：http://www.ranknow.cn/tools/redirectcheck监测是否成功重定向