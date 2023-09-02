---
title: 修改Nginx的header伪装服务器
author: admin
type: post
date: 2010-12-27T07:23:48+00:00
url: /archives/7278
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx

---

有时候为了伪装自己的真实服务器环境.

不像让对方知道自己的webserver真实环境,就不得不修改我们的webserer软件了!

今天看了一下baidu.com的webserver感觉像是nginx修改的.

>

> C:curl-7.18.0>curl.exe -I www.baidu.com
>

>
>

> HTTP/1.1 200 OK
>

>
>

> Date: Tue, 11 Mar 2008 05:00:39 GMT
>

>
>

> Server: BWS/1.0
>

>
>

> Content-Length: 3022
>

>
>

> Content-Type: text/html
>

>
>

> Cache-Control: private
>

>
>

> Expires: Tue, 11 Mar 2008 05:00:39 GMT
>

>
>

> Set-Cookie: BAIDUID=41BB2845D3E8BC1AEE99D4CECB90C50A:FG=1; expires=Tue, 11-
>

>
>

> 8 05:00:39 GMT; path=/; domain=.baidu.com
>

>
>

> P3P: CP=” OTI DSP COR IVA OUR IND COM “
>

哈只是感觉,人家有坚强的开发后盾,可能是自己开发的!

于是自己翻了一下nginx源码了,发现竟然很容修改就可以实现.

>

> cd /usr/local/src/nginx-0.5.35/src/core/
>

>
>

> [root@zyatt core]# cat nginx.h
>

>
>

> /*
>

>
>

> * Copyright (C) Igor Sysoev
>

>
>

> */
>

>
>

> #ifndef _NGINX_H_INCLUDED_
>

>
>

> #define _NGINX_H_INCLUDED_
>

>
>

> #define NGINX_VERSION      “1.0”
>

>
>

> #define NGINX_VER          “LPKWS/” NGINX_VERSION
>

>
>

> #define NGINX_VAR          “LPKWS”
>

>
>

> #define NGX_OLDPID_EXT     “.oldbin”
>

>
>

> #endif /* _NGINX_H_INCLUDED_ */
>

测试效果

>

> C:curl-7.18.0>curl.exe -I 211.100.11.122/info.php  (此Nginx没有做优化,配置expires,gzip等,仅为测试)
>

>
>

> HTTP/1.1 200 OK
>

>
>

> Server: LPKWS/1.0
>

>
>

> Date: Tue, 11 Mar 2008 04:53:02 GMT
>

>
>

> Content-Type: text/html
>

>
>

> Transfer-Encoding: chunked
>

>
>

> Connection: keep-alive
>

>
>

> Keep-Alive: timeout=20
>

>
>

> X-Powered-By: PHP/5.2.4
>

哈,仅为娱乐,要想真正的优化和安全考虑,还是应该好好读读源代码,踏踏实实做好细节工作!