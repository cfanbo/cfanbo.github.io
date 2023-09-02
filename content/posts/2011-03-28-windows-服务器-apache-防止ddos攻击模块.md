---
title: windows 服务器 Apache 防止ddos攻击模块
author: admin
type: post
date: 2011-03-28T08:05:42+00:00
url: /archives/8182
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - DDOS

---

为了防HTTP DoS或DDos攻击，我们可能会对服务器添加很多种防护产品，可能会购买专业的DDoS硬件防火墙，当然，目前并没有一种很成熟的技术能完全封锁住DDoS攻击。但如果对于小型网站服务器来说，Apache的evasive模块是比较简单的处理方法，原理也很简单，判断一段时间内，某个IP访问的次数是否过快，如果过快，就返回403错误。

但是官方的evasive模块发布的是源代码和linux下的RPM压缩包，虽然可以在windows使用源代码编译出这个模块来，但是由于windows系统本身的原因，几乎不会在默认的情况下安装C语言的编译环境，如果需要安装这个编译环境要安装非常多而繁杂的软件，操作起来非常不便。但是在LINUX系统下编译好的文件却不能在WINDOWS下使用，这是两个系统核心的区别，肯定不能使用。

我在别的网站找到了WINDOWS下用的编译好的DLL文件，方便使用WINDOWS系统，同时又是Apache 2.2服务器软件的站长们使用。

**安装方法：**
1、下载附件中的压缩包，解压并拷贝mod_dosevasive22.dll到Apache安装目录下的modules目录（当然也可以是其他目录，需要自己修改路径）。
2、修改Apache的配置文件http.conf。

添加以下内容

LoadModule dosevasive22\_module modules/mod\_dosevasive22.dll
DOSHashTableSize 3097
DOSPageCount 3
DOSSiteCount 50
DOSPageInterval 1
DOSSiteInterval 1
DOSBlockingPeriod 10

其中DOSHashTableSize 3097 记录黑名单的尺寸
DOSPageCount 3 每个页面被判断为dos攻击的读取次数
DOSSiteCount 50 每个站点被判断为dos攻击的读取部件(object)的个数
DOSPageInterval 1 读取页面间隔秒
DOSSiteInterval 1 读取站点间隔秒
DOSBlockingPeriod 10 被封时间间隔秒

\___\___\___\___\___\___\___\___\___\___\___\___\___\____

首先，在 httpd.conf 加入

>

```
LoadModule dosevasive22_module modules/mod_dosevasive22.dll

```

如果需要配置，在 httpd.conf 加入:

>
> DOSHashTableSize 3097
> DOSPageCount 2
> DOSSiteCount 50
> DOSPageInterval 1
> DOSSiteInterval 1
> DOSBlockingPeriod 10
>

```

```

相关下载:

[dosevasive.zip](http://www.lifei.com.cn/upload/dosevasive.zip)

各参数的配置说明如下：

DOSHashTableSize
—————-
The hash table size defines the number of top-level nodes for each child’s
hash table. Increasing this number will provide faster performance by
decreasing the number of iterations required to get to the record, but
consume more memory for table space. You should increase this if you have
a busy web server. The value you specify will automatically be tiered up to
the next prime number in the primes list (see mod_evasive.c for a list
of primes used).

DOSPageCount
————
This is the threshhold for the number of requests for the same page (or URI)
per page interval. Once the threshhold for that interval has been exceeded,
the IP address of the client will be added to the blocking list.

DOSSiteCount
————
This is the threshhold for the total number of requests for any object by
the same client on the same listener per site interval. Once the threshhold
for that interval has been exceeded, the IP address of the client will be added
to the blocking list.

DOSPageInterval
—————
The interval for the page count threshhold; defaults to 1 second intervals.

DOSSiteInterval
—————
The interval for the site count threshhold; defaults to 1 second intervals.

DOSBlockingPeriod
—————–
The blocking period is the amount of time (in seconds) that a client will be
blocked for if they are added to the blocking list. During this time, all
subsequent requests from the client will result in a 403 (Forbidden) and
the timer being reset (e.g. another 10 seconds). Since the timer is reset
for every subsequent request, it is not necessary to have a long blocking
period; in the event of a DoS attack, this timer will keep getting reset.

WHITELISTING IP ADDRESSES
IP addresses of trusted clients can be whitelisted to insure they are never
denied. The purpose of whitelisting is to protect software, scripts, local
searchbots, or other automated tools from being denied for requesting large
amounts of data from the server. Whitelisting should \*not\* be used to add
customer lists or anything of the sort, as this will open the server to abuse.
This module is very difficult to trigger without performing some type of
malicious attack, and for that reason it is more appropriate to allow the
module to decide on its own whether or not an individual customer should be
blocked.

To whitelist an address (or range) add an entry to the Apache configuration
in the following fashion:

DOSWhitelist 127.0.0.1
DOSWhitelist 127.0.0.*

Wildcards can be used on up to the last 3 octets if necessary. Multiple
DOSWhitelist commands may be used in the configuration.