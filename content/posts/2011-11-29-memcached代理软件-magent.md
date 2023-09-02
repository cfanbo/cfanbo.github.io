---
title: Memcached代理软件 magent
author: admin
type: post
date: 2011-11-29T02:23:53+00:00
url: /archives/12152
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - magent
 - memcached

---
magent是一款开源的Memcached代理服务器软件。
**命令参数：**

>

```
-h this message
-u uid
-g gid
-p port, default is 11211. (0 to disable tcp support)
-s ip:port, set memcached server ip and port
-b ip:port, set backup memcached server ip and port
-l ip, local bind ip address, default is 0.0.0.0
-n number, set max connections, default is 4096
-D don't go to background
-k use ketama key allocation algorithm
-f file, unix socket path to listen on. default is off
-i number, max keep alive connections for one memcached server, default is 20
-v verbose
```

**使用方法:**

> magent -s 10.1.2.1 -s 10.1.2.2:11211 -b 10.1.2.3:14000

另外有一个java版的memcache session manager的管理软件．主要用在tomcat的． [https://code.google.com/p/memcached-session-manager/](https://code.google.com/p/memcached-session-manager/)

另外还有一个[[转]Twemproxy——针对MemCached与Redis的代理][1]

 [1]: http://blog.csdn.net/heiyeshuwu/article/details/16105275