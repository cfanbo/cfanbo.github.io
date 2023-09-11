---
title: 'nginx: [warn] the “log_format” directive may be used only on “http” level ．．．解决办法'
author: admin
type: post
date: 2012-05-04T16:50:01+00:00
url: /archives/12842
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx

---
新开了一个vps,装了最新的nginx 1.0.2版本，将原来的虚拟主机配置直接拿过来．用nginx -t 测试语法的时候，发现提示以下警告信息

> [root@centos nginx]# ./sbin/nginx -t
> nginx: [warn] the “log_format” directive may be used only on “http” level in /usr/local/nginx/conf/vhosts/bbs.conf:62

解决办法如下：

将/usr/local/nginx/conf/nginx.conf 里server段里的下面代码移出放到该server段的前面即可。

> log\_format  access  ‘$remote\_addr – $remote\_user [$time\_local] “$request” ‘
> ‘$status $body\_bytes\_sent “$http_referer” ‘
> ‘”$http\_user\_agent” $http\_x\_forwarded_for’;

如果有其的虚拟主机开启了日志，也按上面的要求移出server段放在server段的前面即可。

再**/usr/local/nginx/sbin/nginx -t** 测试一下，没有warn警告信息了。

> [root@centos vhosts]# ../../sbin/nginx -t
> nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
> nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
> [root@centos vhosts]#