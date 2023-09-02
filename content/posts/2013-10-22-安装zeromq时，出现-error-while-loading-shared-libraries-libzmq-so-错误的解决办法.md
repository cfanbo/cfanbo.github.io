---
title: '安装zeromq时，出现 error while loading shared libraries: libzmq.so 错误的解决办法'
author: admin
type: post
date: 2013-10-22T11:42:31+00:00
url: /archives/14617
categories:
 - 系统架构
tags:
 - zeromq

---
Is this on Ubuntu? You’ll need to add /usr/local/lib to ldconfig to be able to use ZeroMQ. Here’s a web page with some info: [http://ubuntuforums.org/showthread.php?t=420008](http://ubuntuforums.org/showthread.php?t=420008)

Here are the actual instructions:

Add **/usr/local/lib** to a new line in ld.so.conf:

```
$ sudo vi /etc/ld.so.conf
```

Rerun ldconfig:

```
$ sudo ldconfig
```

That should work (if I remember correctly). Let me know if you have any issues.