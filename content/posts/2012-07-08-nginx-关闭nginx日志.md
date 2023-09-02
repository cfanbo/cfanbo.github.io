---
title: Nginx——关闭Nginx日志
author: admin
type: post
date: 2012-07-08T07:20:45+00:00
url: /archives/13119
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx

---

有时候，nginx日志十分吓人，我们有个客户受到攻击，nginx出现too many connections错误，日志5分钟就写入了10GB，硬盘很快就会满了。 那么，如何关闭Nginx日志？怎么取消/停止Nginx日志？ 可以修改nginx.conf


```
access_log /dev/null;
error_log /dev/null;
```

这样全部把他们丢到系统的黑洞里了。不用每时每刻都往系统磁盘疯狂的读写日志了 还延长硬盘的寿命。

修改完，重启Nginx( kill -HUP `cat logs/nginx.pid` )即可。