---
title: cacti 监控nginx 出现 no (LWP::UserAgent not found)的解决办法
author: admin
type: post
date: 2011-04-16T16:55:20+00:00
url: /archives/9272
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - CACTI
 - nginx

---
在上一篇<< [如何安装cacti监控nginx的插件](http://blog.haohtml.com/archives/9288) >>文章里我们介绍了如何安装,但在最后发现执行

> get\_nginx\_clients\_status.pl http://nginx.server.tld/nginx\_status

的时候,提示  no (LWP::UserAgent not found) 错误的原因是该系统 perl 缺少了相关组建，解决办法：

```
[shell]yum -y install perl-libwww-perl[/shell]
```

如果还不行，再按下面的方法解决:

```
[shell]#perl -MCPAN -e shell 一直回车，知道出现cpan&amp;gt;&nbsp; 提示符开始。

cpan&gt; install LWP::UserAgent
cpan&gt; exit
[/shell]
```

如果perl相关组件安装成功，就能正常显示了。

[root@localhost scripts]# ./get\_nginx\_clients_status.pl
nginx\_active:149 nginx\_reading:6 nginx\_writing:137 nginx\_waiting:6