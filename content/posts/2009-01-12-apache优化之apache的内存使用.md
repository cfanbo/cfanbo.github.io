---
title: APACHE优化之apache的内存使用
author: admin
type: post
date: 2009-01-12T10:04:40+00:00
excerpt: |
 Apache是运行在Linux操作系统上的头号Web服务器。很多小地方都可以用来调整Apache的性能，并降低它对系统资源的影响。其中一个就是调整内存使用率，当然达到这一目的可能还是需要花点功夫的。
 例如，通过ps来确定httpd线程的内存使用率，可以输入下面的命令：
url: /archives/868
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache

---
Apache是运行在Linux操作系统上的头号Web服务器。很多小地方都可以用来调整Apache的性能，并降低它对系统资源的影响。其中一个就是调整内存使用率，当然达到这一目的可能还是需要花点功夫的。
例如，通过ps来确定httpd线程的内存使用率，可以输入下面的命令：
\# ps -U apache -u apache u

USERPID %CPU %MEMVSZRSS TTYSTAT START TIME COMMAND
apache130670.05.3 149704 54504 ?SOct071:53 /usr/sbin/httpd -f /etc/httpd/conf/httpd.conf -DAPACHE2
…

上面这段输出显示了单个httpd进程使用了50 MB的RSS（驻留集大小）内存（或者非交换物理内存），以及149 MB的VSZ（虚拟）内存。这当然在很大程度上取决于你在Apache里加载和运行的模块数量。这决不是一个固定的数字。由于这个数字里还包含了共享库包，所以不是100％的准确。我们可以认为RSS数字的一半是httpd线程真正使用的内存数，这可能还有点保守，但是离我们的目的已经非常接近了。

在本文里，我们假设每个httpd进程都在使用了27 MB内存。然后，你需要确定可以让httpd真正使用的内存数。根据运行在机器上的其他进程，你可能希望要求50％的物理内存都供Apache使用。在一个装有1GB内存的系统上，就有512MB的内存可以被划分为多个27MB的内存，也就是大约19个并发的httpd内存。有些人坚持认为每个httpd 线程“真正”使用大约5MB的内存，所以从理论上讲你可以把512MB的内存划分出102个并发进程供Apache使用（要记住的是，除非你的网站需要极其巨大的流量，否则这种情况是非常罕见的）。
在默认状态下，Apache会分配最大256个并发客户端连接，或者256个进程（每一个都对应一个请求）。按照这种设置，一个流量巨大的网站会在顷刻间崩溃（即使你假设每个进程占用5MB内存，那也需要1.3GB的内存来满足请求的数量）。如果不采取其它措施，系统会通过硬盘来尝试使用交换空间以处理它无法在物理内存中完成的任务。

**其他可以调整的项目**包括KeepAlive、KeepAliveTimeout和MaxKeepAliveRequests等设置。可以放在httpd.conf文件里的推荐设置有：

ServerLimit 128MaxClients 128KeepAlive OnKeepAliveTimeout 2MaxKeepAliveRequests 100

通过将KeepAliveTimeout从15秒减到2秒，可以增加MaxClients命令；19太小，而128要好得多。通过减少进程存活的秒数，你可以在相同的时间内允许更多的连接。

当然，如果没有真正的测试在背后支持，数字就是毫无意义的，这就是ab的作用之所在。使用ab对Apache配置文件（MaxClients 等于 256、ServerLimit等于256、KeepAliveTimeout等于15）进行调整，使其能够满足1000个请求（100个连续请求并发产生）的调整方法如下。（在执行测试的时候要确保服务器上有一个终端打开以观察系统的负载。）
$ ab -n 1000 -c 100 -k http://yoursite.com/index.php

现在把上面的服务器设置改为更加保守的设置，重新启动Apache，试着再次测试（总是从远程计算机上进行，而不是本机）。
在这里的测试中，不同的设置导致执行所消耗的时间产生了一倍的差距（分别为27.8s和16.8s），但是负载的平均值为0.03和 0.30。这可能会使得你的网站变得稍慢，但是会确保它不会在高负载的情况下崩溃。还要记住的是，你将需要进行多次测试，以便取得一个平均值。

使用ab是测试调整Apache配置的一个极佳方法，应该在你每次做出影响性能的更改时使用它