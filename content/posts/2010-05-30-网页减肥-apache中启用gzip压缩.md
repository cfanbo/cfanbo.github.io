---
title: 网页减肥-apache中启用gzip压缩
author: admin
type: post
date: 2010-05-30T13:12:41+00:00
url: /archives/3723
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - apache

---
先启用 LoadModule deflate\_module modules/mod\_deflate.so,只需要把前面的#去掉就可以了。

然后在httpd.conf最下面添加以下行:

DeflateBufferSize 8096
DeflateCompressionLevel 1
DeflateMemLevel 9
DeflateWindowSize 15

DeflateFilterNote Input instream
DeflateFilterNote Output outstream
DeflateFilterNote Ratio ratio
DeflateFilterNote ratio
LogFormat ‘”%r” %{outstream}n/%{instream}n (%{ratio}n%%)’ deflate
CustomLog logs/deflate.log deflate


SetOutputFilter DEFLATE

AddOutputFilterByType DEFLATE text/html text/css application/x-javascript text/plain text/xml

然后重启apache,即可。可以用firefox的插件yslow来查看效果，此时会看到”Compress components with gzip”项的等级为A,说明配置成功了.