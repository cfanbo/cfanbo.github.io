---
title: Nginx中的gzip模块简介
author: admin
type: post
date: 2011-09-02T09:46:03+00:00
url: /archives/11211
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - gzip
 - nginx

---
## gzip

**语法:** _gzip on|off_

**默认值:** _gzip off_

**作用域:** _http, server, location, if (x) location_

开启或者关闭gzip模块



## gzip_buffers

**语法:** _gzip_buffers number size_

**默认值:** _gzip_buffers 4 4k/8k_

**作用域:** _http, server, location_
设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流。例如 4 4k 代表以4k为单位，按照原始数据大小以4k为单位的4倍申请内存。 4 8k 代表以8k为单位，按照原始数据大小以8k为单位的4倍申请内存。

如果没有设置，默认值是申请跟原始数据相同大小的内存空间去存储gzip压缩结果。



## **gzip\_comp\_level**

**语法:** _**gzip\_comp\_level** 1..9_

**默认值:** _**gzip\_comp\_level** 1_

**作用域:** _http, server, location_

gzip压缩比，1 压缩比最小处理速度最快，9 压缩比最大但处理最慢（传输快但比较消耗cpu）。



## gzip\_min\_length

**语法:** _gzip\_min\_length length_

**默认值:** _gzip\_min\_length 0_

**作用域:** _http, server, location_
设置允许压缩的页面最小字节数，页面字节数从header头中的Content-Length中进行获取。

默认值是0，不管页面多大都压缩。

建议设置成大于1k的字节数，小于1k可能会越压越大。即: gzip\_min\_length 1024



## gzip\_http\_version

**语法:** _gzip\_http\_version 1.0|1.1_

**默认值:** _gzip\_http\_version 1.1_

**作用域:** _http, server, location_

识别http的协议版本。由于早期的一些浏览器或者http客户端，可能不支持gzip自解压，用户就会看到乱码，所以做一些判断还是有必要的。 注：21世纪都来了，现在除了类似于百度的蜘蛛之类的东西不支持自解压，99.99%的浏览器基本上都支持gzip解压了，所以可以不用设这个值,保持系 统默认即可。



## gzip_proxied

**语法:** _gzip\_proxied [off|expired|no-cache|no-store|private|no\_last\_modified|no\_etag|auth|any] …_

**默认值:** _gzip_proxied off_

**作用域:** _http, server, location_

Nginx作为反向代理的时候启用，开启或者关闭后端服务器返回的结果，匹配的前提是后端服务器必须要返回包含”Via”的 header头。

 * off – 关闭所有的代理结果数据的压缩
 * expired – 启用压缩，如果header头中包含 “Expires” 头信息
 * no-cache – 启用压缩，如果header头中包含 “Cache-Control:no-cache” 头信息
 * no-store – 启用压缩，如果header头中包含 “Cache-Control:no-store” 头信息
 * private – 启用压缩，如果header头中包含 “Cache-Control:private” 头信息
 * no\_last\_modified – 启用压缩,如果header头中不包含 “Last-Modified” 头信息
 * no_etag – 启用压缩 ,如果header头中不包含 “ETag” 头信息
 * auth – 启用压缩 , 如果header头中包含 “Authorization” 头信息
 * any – 无条件启用压缩



## gzip_types

**语法:** _gzip_types mime-type [mime-type …]_

**默认值:** _gzip_types text/html_

**作用域:** _http, server, location_

匹配MIME类型进行压缩，（无论是否指定）”text/html”类型总是会被压缩的。
注意：如果作为http server来使用，主配置文件中要包含文件类型配置文件

```
http
{
 include       conf/mime.types;
 ......
}
```

如果你希望压缩常规的文件类型，可以写成这个样子

```
http
{
: include       conf/mime.types;

: gzip on;
: gzip_min_length  1000;
: gzip_buffers     4 8k;
: gzip_http_version 1.1;
: gzip_types       application/x-javascript text/css application/xml;

: ......
}
```