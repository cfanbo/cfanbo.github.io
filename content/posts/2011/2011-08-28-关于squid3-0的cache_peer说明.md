---
title: 关于SQUID3.0的cache_peer说明介绍
author: admin
type: post
date: 2011-08-28T09:50:58+00:00
url: /archives/11051
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - Squid

---
http_port 8000 vhost # Squid 服务器监听本机 8000 端口，vhost 支持虚拟主机。

cache_peer 192.168.1.50 parent 81 0 no-query originserver weight=1 name=a
cache_peer 192.168.1.50 parent 82 0 no-query originserver weight=1 name=b
cache_peer 192.168.1.51 parent 80 0 no-query originserver weight=1 name=c

**cache\_peer\_domain a www.serverA1.com**
**cache\_peer\_domain b www.serverA2.com**
**cache\_peer\_domain c www.serverB.com
＃以上六行配置，让 Squid 服务器知道：**

**
＃从客户端过来的请求，如果是 www.serverA1.com，则 Squid 向 ServerA 192.168.1.50 的端口 81发送请求；
＃****从客户端过来的请求，如果是 www.serverA2.com，则 Squid 向 ServerA 192.168.1.50 的端口 82发送请求；**
**＃****从客户端过来的请求，如果是 www.serverB.com，则 Squid 向 ServerA 192.168.1.50 的端口 80发送请求；**

cache\_dir ufs /squid\_cache 256 16 256 ＃指定 Squid 服务器存放数据的目录

acl all src 0.0.0.0/0.0.0.0
http_access allow all

**cache\_peer\_access a allow all**
**cache\_peer\_access b allow all**
**cache\_peer\_access c allow all**
**＃设置访问权限，允许所有外部客户端访问 a b c（我们定义的三个虚拟主机）**

其它配置项默认即可。

**扩展阅读：**

常用 squid 配置及命令 ：

squid的安全设置：

squid日志详解：