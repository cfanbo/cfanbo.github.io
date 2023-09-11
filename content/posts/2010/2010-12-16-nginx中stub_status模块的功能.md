---
title: nginx中stub_status模块的功能
author: admin
type: post
date: 2010-12-16T05:42:10+00:00
url: /archives/6915
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx

---

Nginx中的stub_status模块主要用于查看Nginx的一些状态信息.

本模块默认是不会编译进Nginx的,如果你要使用该模块,则要在编译安装Nginx时指定:

>

> ./configure –with-http_stub_status_module
>

配置示例如代码:

>

> server
>

>
>

> {
>

>
>

> listent 80;
>

>
>

> server_name status.yourdomain.com;
>

>
>

> location / {
>

>
>

> stub_status on;
>

>
>

> access_log off;
>

>
>

> allow 192.168.0.1.2;
>

>
>

> deny all;
>

>
>

> }
>

>
>

> }
>

======================================

语法: **stub_status on**

默认值:None

使用环境:location

该指令用于开启Nginx状态信息

访问以上示例中配置的 http://status.yourdomain.com/,则显示的Nginx状态信息如下:

[![](http://blog.haohtml.com/wp-content/uploads/2010/12/nginx_stub_status.jpg)](http://blog.haohtml.com/wp-content/uploads/2010/12/nginx_stub_status.jpg)

**Active connections:** 对后端发起的活动连接数.

**Server accepts handled requests:** Nginx总共处理了38810620个连接,成功创建38810620次握手(证明中间没有失败的),总共处理了298655730个请求.

**Reading:** Nginx 读取到客户端的Header信息数.

**Writing:** Nginx 返回给客户端的Header信息数.

**Waiting:** 开启keep-alive的情况下,这个值等于 active – (reading + writing),意思就是Nginx已经处理完成,正在等候下一次请求指令的驻留连接.

所以,在访问效率高,请求很快被处理完毕的情况下,Waiting数比较多是正常的.如果reading +writing数较多,则说明并发访问量非常大,正在处理过程中.