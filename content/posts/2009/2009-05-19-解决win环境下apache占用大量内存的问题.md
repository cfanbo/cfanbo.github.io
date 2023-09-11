---
title: 解决win环境下Apache占用大量内存的问题
author: admin
type: post
date: 2009-05-19T01:38:39+00:00
excerpt: |
 |
 我有个服务是在windows下的Apache2提供的。访问量不是很大，隔4、5天竟然停止服务，
 调查发现Apache2的进程httpd.exe占用内存达到了1.5G。在网上找到如下解决办法。

 用记事本打开apache2\conf\httpd.conf，查找MaxRequestsPerChild，将MaxRequestsPerChild 0改成MaxRequestsPerChild 50即可。
 原因是：
url: /archives/1411
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache

---
我有个服务是在windows下的Apache2提供的。访问量不是很大，隔4、5天竟然停止服务，调查发现Apache2的进程httpd.exe占用内存达到了1.5G。在网上找到如下解决办法。

用记事本打开apache2\conf\httpd.conf，查找MaxRequestsPerChild，将MaxRequestsPerChild 0改成MaxRequestsPerChild 50即可。

**原因是：**

　　通常在“Windows任务管理器－进程”中可以看到两个apache.exe进程，一个是父进程、一个是子进程，父进程接到访问请求后，将请 求交由子进程处理。MaxRequestsPerChild这个指令设定一个独立的子进程将能处理的请求数量。在处理 “MaxRequestsPerChild 数字”个请求之后，子进程将会被父进程终止，这时候子进程占用的内存就会释放，如果再有访问请求，父进程会重新产生子进程进行处理。

　　如果MaxRequestsPerChild缺省设为0(无限)或较大的数字(例如10000以上)可以使每个子进程处理更多的请求，不会因为 不断终止、启动子进程降低访问效率，但MaxRequestsPerChild设置为0时，如果占用了200～300M内存，即使负载下来时占用的内存也 不会减少。内存较大的服务器可以设置为0或较大的数字。内存较小的服务器不妨设置成30、50、100，以防内存溢出。

因为Windows NT下Apache只能启动父子两个进程，因此只能通过增大单个进程的线程数以及单个进程能够处理的最大请求数来进行优化。其他优化的参数同Linux系统下是一样的，大家可以加以参考。下面针对上述两个参数给出一个建议的设置：

    ThreadsPerChild 250
    MaxRequestsPerChild 5000



来源：

1. http://mingling123456.blog.163.com/blog/static/1066189200814112544921/

2. http://www.javatang.com/archives/2008/02/19/0801260.html