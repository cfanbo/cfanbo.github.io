---
title: apache中对设置网页的Expires头-seo篇
author: admin
type: post
date: 2010-04-07T07:12:50+00:00
url: /archives/3313
IM_contentdowned:
 - 1
categories:
 - 服务器

---
平时我用一般用Yslow这个ff下的插件来检查网页的好坏,其中有一项为添加文件过期头.

实施这一方法将节省你难以置信数额的带宽，极大地加快你的网站为你的网站访客。基本上，对于图片，CSS ， JavaScript以及其他文件可以通过优化更快的下载，告诉你的网站访问者快取记忆体，为他们在某一段时间内。默认的行为是每一次请求检查文件的 last-modified 和/或者 Etag headers。
所以一个用户去/home/index.html，及浏览器缓存所有图象和文件。然后用户离开网站稍后回来，与浏览器发送If-Modified- Since 有条件的GET 请求为每一个缓存的项目时，基本上看，如果文件已被改变和他们必须更新他们的缓存。


当你执行在这篇文章中所述的缓存方法，你可以指定某文件或扩展名被缓存为某一特定数额的时间。这些文件然后缓存在你的网站访客和他们不发送If- Modified-Since头直到设置的缓存时间已经到了。
#================================================= ============================#
\# TIME CHEAT SHEET
#================================================= ============================#
\# 300 5 M # 604800 1 W
\# 2700 45 M # 1814400 3 W
\# 3600 1 H # 2419200 1 M
\# 54000 15 H # 14515200 6 M
\# 86400 1 D # 26611200 11 M
\# 518400 6 D # 29030400 1 Y (never expire)

mod_expires 模块的主要作用是自动生成页面头部信息中的 Expires 标签和 Cache-Control 标签，从而降低客户端的访问频率和次数，达到减少不必要流量和增加访问速度的目的。

mod_expires 是 apache 众多模块中配置比较简单的一个，它一共只有三条指令：

ExpiresActive
ExpiresByType
ExpiresDefault
ExpiresActive 指令：打开或关闭产生”Expires:”和”Cache-Control:”头的功能。

ExpiresByType 指令：指定MIME类型的文档（例如：text/html）的过期时间。
ExpiresDefault 指令：默认所有文档的过期时间。

过期时间的写法：

“access plus 1 month”
“access plus 4 weeks”
“now plus 30 days”
“modification plus 5 hours 3 minutes”
A2592000
M604800
access、now及A 三种写法的意义相同，指过期时间从访问时开始计算。
modification及M 的意义相同，指过期时间是以被访问文件的最后修改时间开始计算。
所以，后一种写法只对静态文件起作用，而由脚本生成的动态页面不受它的作用。
如下：
 **第一个解决办法是Apache模块mod_expires 1.3 2.0 2.2**
ExpiresActive On
ExpiresDefault A300
ExpiresByType image/x-icon A2592000
ExpiresByType application/x-javascript A2592000
ExpiresByType text/css A2592000
ExpiresByType image/gif A604800
ExpiresByType image/png A604800
ExpiresByType image/jpeg A604800
ExpiresByType text/plain A604800
ExpiresByType application/x-shockwave-flash A604800
ExpiresByType video/x-flv A604800
ExpiresByType application/pdf A604800
ExpiresByType text/html A300
 **第二个解决办法是mod_headers 1.3 2.0 2.2**
\# YEAR

Header set Cache-Control “max-age=2592000″

\# WEEK

Header set Cache-Control “max-age=604800″

\# NEVER CACHE

Header set Expires “Thu, 01 Dec 2003 16:00:00 GMT”
Header set Cache-Control “no-store, no-cache, must-revalidate”
Header set Pragma “no-cache”

注：用filesmatch和files在htaccess文件
这里是Headers当下载一个JPEG图像的时候，
这个缓存方案实施后和没有缓存时的效果。
JPEG 没有缓存的时
Last-Modified: Wed, 22 Feb 2006 12:16:56 GMT
ETag: “b57d54-45e7″
Accept-Ranges: bytes
Content-Length: 17895
Connection: close
Content-Type: image/jpeg
缓存过的
Cache-Control: max-age=2592000
Expires: Tue, 28 Mar 2006 16:23:52 GMT
Last-Modified: Wed, 22 Feb 2006 12:16:56 GMT
ETag: “b57d54″
Accept-Ranges: bytes
Content-Length: 17895
Connection: close
Content-Type: image/jpeg
Content-Language: en
附：
apache配置文件例子：
example 1
\# htm files are php
AddHandler application/x-httpd-php .php .htm
\# setup errordocuments to local php file
ErrorDocument 404 /cgi-bin/error.htm
ErrorDocument 403 /cgi-bin/error.htm
ErrorDocument 500 /cgi-bin/error.htm
\# Turn on Expires and set default expires to 3 days
ExpiresActive On
ExpiresDefault A259200
\# Set up caching on media files for 1 month

ExpiresDefault A2419200
Header append Cache-Control “public”

\# Set up 2 Hour caching on commonly updated files

ExpiresDefault A7200
Header append Cache-Control “private, must-revalidate”

\# Force no caching for dynamic files

ExpiresDefault A0
Header set Cache-Control “no-store, no-cache, must-revalidate, max-age=0″
Header set Pragma “no-cache”

example 2
\# htm files are php
AddHandler application/x-httpd-php .php .htm
\# setup errordocuments to local php file
ErrorDocument 404 /cgi-bin/error.htm
ErrorDocument 403 /cgi-bin/error.htm
ErrorDocument 500 /cgi-bin/error.htm
\# Turn on Expires and set default to 0
ExpiresActive On
ExpiresDefault A0
\# Set up caching on media files for 1 year (forever?)

ExpiresDefault A29030400
Header append Cache-Control “public”

\# Set up caching on media files for 1 week

ExpiresDefault A604800
Header append Cache-Control “public, proxy-revalidate”

\# Set up 2 Hour caching on commonly updated files

ExpiresDefault A7200
Header append Cache-Control “private, proxy-revalidate, must-revalidate”

\# Force no caching for dynamic files

ExpiresDefault A0
Header set Cache-Control “no-cache, no-store, must-revalidate, max-age=0, proxy-revalidate, no-transform”
Header set Pragma “no-cache”

需要提醒您的是，经过Expires处理的URL地址，可以被浏览器及代理服务器缓存，请只在需要的地方使用。

参考文档：
Apache 1.3.x
Apache 2.0.x