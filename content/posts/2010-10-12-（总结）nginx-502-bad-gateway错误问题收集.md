---
title: （总结）Nginx 502 Bad Gateway错误问题收集
author: admin
type: post
date: 2010-10-12T01:47:52+00:00
url: /archives/5984
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx

---
nginx和lighttpd的文档真的很少，更不用说中文文档了，所以收集一些和502有关的错误在这里。

502是FastCGI出现问题，所以从FastCGI配置入手。

**1.请检查你的FastCGI进程是否启动**

**2.FastCGI进程不够使用**
请通过执行 netstat -anpo | grep “php-cgi” | wc -l 判断，是否接近你启动的FastCGI进程，接近你的设置，表示进程不够

**3.执行超时**
请把
fastcgi\_connect\_timeout 300;
fastcgi\_send\_timeout 300;
fastcgi\_read\_timeout 300;
这几项的值调高

**4.FastCGI缓冲不够**
nginx和apache一样，有前端缓冲限制
请把
fastcgi\_buffer\_size 32k;
fastcgi_buffers 8 32k;
这几项的值调高

**5.Proxy缓冲不够**
如果你使用了Proxying，请把
proxy\_buffer\_size  16k;
proxy_buffers      4 16k;
这几项的值调高

**6.https转发配置错误**
正确的配置方法
server_name www.mydomain.com;

location /myproj/repos {

set $fixed\_destination $http\_destination;
if ( $http_destination ~\* ^https(.\*)$ )
{
set $fixed_destination http$1;
}

proxy\_set\_header        Host $host;
proxy\_set\_header        X-Real-IP $remote_addr;
proxy\_set\_header        Destination $fixed_destination;
proxy\_pass              http://subversion\_hosts;
}

**7.php脚本执行时间过长**
将php-fpm.conf的0s的0s改成一个时间。