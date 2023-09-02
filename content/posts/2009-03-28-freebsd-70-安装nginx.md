---
title: FreeBSD 7.0 安装Nginx
author: admin
type: post
date: 2009-03-28T05:19:11+00:00
url: /archives/1149
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx

---
来源： [http://bbs.chinaunix.net/viewthread.php?tid=1039563&extra=&page=1](http://bbs.chinaunix.net/viewthread.php?tid=1039563&extra=&page=1)

#/usr/ports/www/nginx

#make config

lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk

x                     Options for nginx 0.5.34                       x

x lqqqqq^(-)qqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk x

x x [X] HTTP_ADDITION_MODULE  Enable http_addition module          x x

x x [X] HTTP_DAV_MODULE       Enable http_webdav module            x x

x x [X] HTTP_FLV_MODULE       Enable http_flv module               x x

x x [X] HTTP_PERL_MODULE      Enable http_perl module              x x

x x [X] HTTP_REALIP_MODULE    Enable http_realip module            x x

x x [X] HTTP_REWRITE_MODULE   Enable http_rewrite module           x x

x x [X] HTTP_SSL_MODULE       Enable http_ssl module               x x

x x [X] HTTP_STATUS_MODULE    Enable http_stub_status module       x x

x x [X] HTTP_SUB_MODULE       Enable http_sub module               x x

x x [ ] MAIL_MODULE           Enable IMAP4/POP3/SMTP proxy module  x x

x x [ ] MAIL_IMAP_MODULE      Enable IMAP4 proxy module            x x

x x [ ] MAIL_POP3_MODULE      Enable POP3 proxy module             x x

x x [ ] MAIL_SMTP_MODULE      Enable SMTP proxy module             x x

x x [ ] MAIL_SSL_MODULE       Enable mail_ssl module               x x

x x [X] WWW                   Enable html sample files             x x

tqmqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqjqu

x                       [  OK  ]       Cancel                        x

mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj

#make install clean