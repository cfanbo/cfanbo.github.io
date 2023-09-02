---
title: nginx 502 bad gateway 解决办法
author: admin
type: post
date: 2009-08-26T08:44:09+00:00
excerpt: |
 NGINX 502 Bad Gateway错误是FastCGI有问题，造成NGINX 502错误的可能性比较多。将网上找到的一些和502 Bad Gateway错误有关的问题和排查方法列一下，先从FastCGI配置入手：

 1.FastCGI进程是否已经启动

 2.FastCGI worker进程数是否不够
 运行 netstat -anpo | grep “php-cgi” | wc -l 判断是否接近FastCGI进程，接近配置文件中设置的数值，表明worker进程数设置太少
 参见：http://blog.s135.com/post/361.htm
url: /archives/2288
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx

---
NGINX 502 Bad Gateway错误是FastCGI有问题，造成NGINX 502错误的可能性比较多。将网上找到的一些和502 Bad Gateway错误有关的问题和排查方法列一下，先从FastCGI配置入手：

**1.FastCGI进程是否已经启动**

**2.FastCGI worker进程数是否不够**
运行 netstat -anpo | grep “php-cgi” | wc -l 判断是否接近FastCGI进程，接近配置文件中设置的数值，表明worker进程数设置太少
通过命令查看服务器上一共开了多少的 php-cgi 进程

> ps -fe |grep “php” | grep -v “grep” | wc -l

使用如下命令查看已经有多少个php-cgi进程用来处理tcp请求

> netstat -anop | grep “php” | grep -v “grep” | wc -l

接近配置文件中设置的数值，表明worker进程数设置太少
参见： [http://blog.s135.com/post/361.htm](http://blog.s135.com/post/361.htm)

**3.FastCGI执行时间过长**
根据实际情况调高以下参数值

> fastcgi\_connect\_timeout 300;
> fastcgi\_send\_timeout 300;
> fastcgi\_read\_timeout 300;

**4.FastCGI Buffer不够**
nginx和apache一样，有前端缓冲限制，可以调整缓冲参数

> fastcgi\_buffer\_size 32k;
> fastcgi_buffers 8 32k;

**5.Proxy Buffer不够**
如果你用了Proxying，调整

> proxy\_buffer\_size  16k;
> proxy_buffers      4 16k;

参见： [http://www.ruby-forum.com/topic/169040](http://www.ruby-forum.com/topic/169040)

**6.https转发配置错误**
正确的配置方法

> server_name www.mydomain.com;
> location /myproj/repos {
> set $fixed\_destination $http\_destination;
> if ( $http_destination ~\* ^https(.\*)$ )
> {
> set $fixed_destination http$1;
> }
> proxy\_set\_header Host $host;
> proxy\_set\_header X-Real-IP $remote_addr;
> proxy\_set\_header Destination $fixed_destination;
> proxy\_pass http://subversion\_hosts;
> }

参见： [http://www.ruby-forum.com/topic/169040](http://www.ruby-forum.com/topic/169040)

7. Nginx和PHP的通讯方式不一致也会引起此502错误

见： [http://blog.haohtml.com/archives/15555](http://blog.haohtml.com/archives/15555)

当然，还要看你后端用的是哪种类型的FastCGI，我用过的有php-fpm，流量约为单台机器500万PV(动态页面), 现在基本上没有碰到502。