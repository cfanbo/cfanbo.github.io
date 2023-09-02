---
title: 网页优化-apache中文件有效期的设置-yslow
author: admin
type: post
date: 2010-05-30T14:43:40+00:00
url: /archives/3728
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - apache
 - 网页优化

---
前面我用已经启用了网页压缩功能，见 [http://blog.haohtml.com/index.php/archives/3723](http://blog.haohtml.com/index.php/archives/3723),下面我们来对网页元素有效期进行设置。

首先，启用LoadModule expires\_module modules/mod\_expires.so,只要在httpd.conf中把前面的#号去掉就可以了。然后在httpd.conf最后添加以下几行

ExpiresActive On

ExpiresDefault “access plus 10 years”

重启apache，可以用firefox浏览器中的yslow插件查看最终效果，此时”add expires haders”项应该为A。表示配置成功.