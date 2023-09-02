---
title: nginx location基础
author: admin
type: post
date: 2010-11-12T07:45:03+00:00
url: /archives/6626
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - location
 - nginx

---
**基本语法**

location [=|~|~*|^~] /uri/ { … }

= 严格匹配。如果这个查询匹配，那么将停止搜索并立即处理此请求。

~ 为区分大小写匹配

~* 为不区分大小写匹配

!~和!~*分别为区分大小写不匹配及不区分大小写不匹配

^~ 如果把这个前缀用于一个常规字符串,那么告诉nginx 如果路径匹配那么不测试正则表达式。

**例如:**

location = / { # 只匹配 / 查询。

location / { # 匹配任何查询，因为所有请求都已 / 开头。但正则表达式规则和长的块规则将被优先和查询匹配。

location ^~ /images/ { # 匹配任何已 /images/ 开头的任何查询并且停止搜索。任何正则表达式将不会被测试。

location ~* \.(gif|jpg|jpeg)$ { # 匹配任何以 gif、jpg 或 jpeg 结尾的请求。

**++ 文件及目录匹配**

* -f和!-f用来判断是否存在文件

* -d和!-d用来判断是否存在目录

* -e和!-e用来判断是否存在文件或目录

* -x和!-x用来判断文件是否可执行

**++ 一些可用的全局变量**

$args

$content_length

$content_type

$document_root

$document_uri

$host

$http\_user\_agent

$http_cookie

$limit_rate

$request\_body\_file

$request_method

$remote_addr

$remote_port

$remote_user

$request_filename

$request_uri

$query_string

$scheme

$server_protocol

$server_addr

$server_name

$server_port

$uri