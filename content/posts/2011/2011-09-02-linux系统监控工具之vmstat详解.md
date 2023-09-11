---
title: Linux系统监控工具之vmstat详解
author: admin
type: post
date: 2011-09-01T19:58:47+00:00
url: /archives/11185
IM_data:
 - 'a:2:{s:60:"http://images.51cto.com/files/uploadimg/20100519/1023590.jpg";s:67:"http://blog.haohtml.com/wp-content/uploads/2011/09/6140_1023590.jpg";s:60:"http://images.51cto.com/files/uploadimg/20100519/1023591.jpg";s:67:"http://blog.haohtml.com/wp-content/uploads/2011/09/1cb7_1023591.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux
 - vmstat

---
vmstat是一个十分有用的Linux系统监控工具，使用vmstat命令可以得到关于进程、内存、内存分页、堵塞IO、traps及CPU活动的信息。

**一、前言**

很显然从名字中我们就可以知道vmstat是一个查看虚拟内存（Virtual Memory）使用状况的工具，但是怎样通过vmstat来发现系统中的瓶颈呢？在回答这个问题前，还是让我们回顾一下Linux中关于虚拟内存相关内容。

**二、虚拟内存运行原理**

在系统中运行的每个进程都需要使用到内存，但不是每个进程都需要每时每刻使用系统分配的内存空间。当系统运行所需内存超过实际的物理内存，内核会释放某些进程所占用但未使用的部分或所有物理内存，将这部分资料存储在磁盘上直到进程下一次调用，并将释放出的内存提供给有需要的进程使用。

在Linux内存管理中，主要是通过“调页Paging”和“交换Swapping”来完成上述的内存调度。调页算法是将内存中最近不常使用的页面换到磁盘上，把活动页面保留在内存中供进程使用。交换技术是将整个进程，而不是部分页面，全部交换到磁盘上。

分页(Page)写入磁盘的过程被称作Page-Out，分页(Page)从磁盘重新回到内存的过程被称作Page-In。当内核需要一个分页时，但发现此分页不在物理内存中(因为已经被Page-Out了)，此时就发生了分页错误（Page Fault）。

当系统内核发现可运行内存变少时，就会通过Page-Out来释放一部分物理内存。经管Page-Out不是经常发生，但是如果Page-out频繁不断的发生，直到当内核管理分页的时间超过运行程式的时间时，系统效能会急剧下降。这时的系统已经运行非常慢或进入暂停状态，这种状态亦被称作thrashing(颠簸)。

**三、使用vmstat**

**1.用法**

vmstat \[-a\] \[-n\] \[-S unit\] \[delay [ count\]]

vmstat \[-s\] \[-n\] [-S unit]

vmstat \[-m\] \[-n\] [delay [ count]]

vmstat \[-d\] \[-n\] [delay [ count]]

vmstat \[-p disk partition\] \[-n\] [delay [ count]]

vmstat [-f]

vmstat [-V]

> -a：显示活跃和非活跃内存
>
> -f：显示从系统启动至今的fork数量 。引申阅读： [http://www.cnblogs.com/leoo2sk/archive/2009/12/11/talk-about-fork-in-linux.html](http://www.cnblogs.com/leoo2sk/archive/2009/12/11/talk-about-fork-in-linux.html)
>
> -m：显示slabinfo
>
> -n：只在开始时显示一次各字段名称。
>
> -s：显示内存相关统计信息及多种系统活动数量。
>
> delay：刷新时间间隔。如果不指定，只显示一条结果。
>
> count：刷新次数。如果不指定刷新次数，但指定了刷新时间间隔，这时刷新次数为无穷。
>
> -d：显示磁盘相关统计信息。
>
> -p：显示指定磁盘分区统计信息
>
> -S：使用指定单位显示。参数有 k 、K 、m 、M ，分别代表1000、1024、1000000、1048576字节（byte）。默认单位为K（1024 bytes）
>
> -V：显示vmstat版本信息。

**2.使用说明**

例子1：每2秒输出一条结果


[![](http://blog.haohtml.com/wp-content/uploads/2011/09/vmstat-1.jpg)](http://blog.haohtml.com/wp-content/uploads/2011/09/vmstat-1.jpg)

**字段说明：**

Procs（进程）：

r: 运行队列中进程数量


b: 等待IO的进程数量


Memory（内存）：

swpd: 使用虚拟内存大小


free: 可用内存大小


buff: 用作缓冲的内存大小


cache: 用作缓存的内存大小


Swap：

si: 每秒从交换区写到内存的大小


so: 每秒写入交换区的内存大小


IO：（现在的Linux版本块的大小为1024bytes）

bi: 每秒读取的块数


bo: 每秒写入的块数


system(系统)：

in: 每秒中断数，包括时钟中断。


cs: 每秒上下文切换数。


CPU（以百分比表示）：

us: 用户进程执行时间(user time)


sy: 系统进程执行时间(system time)


id: 空闲时间(包括IO等待时间)


wa: 等待IO时间


例子2：显示活跃和非活跃内存


[![](http://blog.haohtml.com/wp-content/uploads/2011/09/linux-vmstat.jpg)](http://blog.haohtml.com/wp-content/uploads/2011/09/linux-vmstat.jpg)

使用-a选项显示活跃和非活跃内存时，所显示的内容除增加inact和active外，其他显示内容与例子1相同。


**字段说明：**

Memory（内存）：

inact: 非活跃内存大小（当使用-a选项时显示）


active: 活跃的内存大小（当使用-a选项时显示）


如果 r经常大于 4 ，且id经常少于40，表示cpu的负荷很重。

如果pi，po 长期不等于0，表示内存不足。

如果disk 经常不等于0， 且在 b中的队列 大于3， 表示 io性能不好。


Linux在具有高稳定性、可靠性的同时，具有很好的可伸缩性和扩展性，能够针对不同的应用和硬件环境调整，优化出满足当前应用需要的最佳性能。因此企业在维护Linux系统、进行系统调优时，了解系统性能分析工具是至关重要的。


对于FreeBSD下的vmstat基本上差不多，但也有一些差异，请参考： [http://blog.haohtml.com/archives/4465](http://blog.haohtml.com/archives/4465)

**扩展阅读：**

linux下的fork的运行机制： [http://www.cnblogs.com/leoo2sk/archive/2009/12/11/talk-about-fork-in-linux.html](http://www.cnblogs.com/leoo2sk/archive/2009/12/11/talk-about-fork-in-linux.html)