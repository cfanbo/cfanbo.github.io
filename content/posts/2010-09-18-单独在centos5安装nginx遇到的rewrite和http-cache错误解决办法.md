---
title: '[教程]Centos5安装nginx教程及遇到的rewrite和HTTP cache错误解决办法'
author: admin
type: post
date: 2010-09-18T14:51:36+00:00
url: /archives/5758
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - nginx

---
有时候，我们需要单独安装nginx，来处理大量的下载请求。单独在Centos5安装nginx遇到的rewrite和HTTP cache错误解决办法：

wget http://nginx.org/download/nginx-0.8.33.tar.gz
tar -zxvf nginx-0.8.33.tar.gz
cd nginx-0.8.33
./configure –prefix=/usr/local/nginx

安装Nginx时报错

> ./configure: error: the HTTP rewrite module requires the PCRE library.

> 安装pcre-devel解决问题
> yum -y install pcre-devel

错误提示：./configure: error: the HTTP cache module requires md5 functions from OpenSSL library.  You can either disable the module by using

–without-http-cache option, or install the OpenSSL library into the system, or build the OpenSSL library statically from the source with nginx by using –with-http\_ssl\_module –with-openssl= options.

解决办法：

> yum -y install openssl openssl-devel

总结：

> yum -y install pcre-devel openssl openssl-devel
>
> ./configure –prefix=/usr/local/nginx
>
> make
>
> make install

一切搞定.如果在make的时候提示命令make:command not found的话，说明没有安装make，安装一下就可能了。命令：

> #yum -y install make