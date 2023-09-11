---
title: squid中HTTP/1.1 501 Method Not Implemented的解决办法
author: admin
type: post
date: 2011-09-06T04:08:47+00:00
url: /archives/11300
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - purge
 - Squid
 - squidclient

---
刚安装的squid,但在用squidclient清除缓存的时候,提示错误:

```
freebsd# ./squidclient -m PURGE -p 80 http://www.testsquid.com/index.html
HTTP/1.1 501 Method Not Implemented
Date: Tue, 28 Jun 2011 23:03:22 GMT
Server: Apache/2.2.19 (FreeBSD) mod_ssl/2.2.19 OpenSSL/0.9.8k DAV/2
Allow: GET,HEAD,POST,OPTIONS,TRACE
Content-Length: 217
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>501 Method Not Implemented</title>
</head><body>
<h1>Method Not Implemented</h1>
<p>PURGE to /index.html not supported.<br />
</p>
</body></html>
```

**解决办法:**

```
freebsd# ./squidclient -m PURGE -h 192.168.0.120 -p 80 http://www.testsquid.com/index.html
HTTP/1.0 404 Not Found
Server: squid/3.0.STABLE25
Mime-Version: 1.0
Date: Tue, 28 Jun 2011 23:05:49 GMT
Content-Length: 0
X-Cache: MISS from cache.testsquid.com
Via: 1.0 cache.testsquid.com (squid/3.0.STABLE25)
Connection: close
```

指定squid监听的IP地址,我这里将squid(80)和apache(81)安装在了一台机器上,为了安全起见让apache只监听了127.0.0.1:81端口.指定ip就可以了