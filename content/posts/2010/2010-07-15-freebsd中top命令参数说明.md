---
title: FreeBSD中top命令参数说明
author: admin
type: post
date: 2010-07-15T09:50:19+00:00
url: /archives/4681
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - top

---
**top监控命令在FreeBSD上的使用**
top监控工具可以显示CPU占用率为前几位的进程，并提供CPU的实时活动情况

语法：top \[-s time\] \[-d count\] \[-q\] \[-h\] \[-n number\] \[-f filename\] \[-o field\]\[-U usename\]
 **-S** 将系统进程信息也显示到屏幕上，默认情况下，top不显示系统进程的信息
 **-b** 使用”batch”方式运行top。在此种方式下，所有来自终端的输入都将被忽略，但交互键(比如^C and ^\)
依然起使用。这是运行top输出到哑终端或输到非终端的默认运行方式
 **-i** 使用交互运行top程序，在此种方式下，命令会被进程立即被处理。不管命令是不是能被top所理解执行，
屏幕都将立即更新。这是top的默认运行方式。
 **-I** 不显示空闲进程，在默认情况下，top连同空闲进程的信息一同输出。
 **-t** 不显示top进程自己
 **-n** 不以交互方式使用top命令，作用同”batch”方式。
 **-s** time 设置屏幕刷新的延时，单位为秒，默认值5秒
 **-d** count 设置屏幕刷新的次数，刷新显示完count次后退出
 **-q** 如果经过nice授权，使用-q可以使top运行的更快一些，这样，在系统反应缓慢的时候，可以会更快的找到存在的问题。


 **此选项在FreeBSD下只有root可以使用**
 **-n** number 设置每一屏幕显示的进程数目，number值超过进程最大数目，则设置无效
 **-u** 用显示User ID代替username，提高命令运行速度
 **-v** 显示程序版本号后，立即退出。如果要在top运行时查看版本号，输入”?”
 **-o** 以指定的字段排序显示进行信息。字段名必须为输入在屏幕的可见列的名字，而且必须是小写。

比如”cpu”、”size”、”res”与”time”,但不同的操作系统间有许多的不同。注意不是每个UNIX操作系统都支持此选项。
-U 只显示属于后面所跟用户名的进程的信息.

 **屏幕控制命令**
交换方式下，可以使用以下命令控制top
^L – 刷新屏幕
q – 退出
h or ? – 显示帮助
d – 修改刷新显示的次数
e – 显示最近”kill”或”renice”命令所产生的错误
i – 显示/不显示处于空闲的进程
I – 作用同 ‘i’
k – kill 进程; 发送一个信号到某个进程列表
n or # – 修改显示进程的数目
o – 以特定的字段排序 (pri, size, res, cpu, time)
r – renice 一个进程
s – 修改输入的更新时间
u – 只显示属于某个用户的进程 (+ selects all users)

顺序显示下面三个常规的信息
 **一． 系统信息:**

> ****last pid: 22228; load averages: 0.25, 0.97, 1.56 up 44+03:25:56 21:39:36
> 274 processes: 3 running, 259 sleeping, 12 zombie
> CPU states: 2.9% user, 0.0% nice, 4.2% system, 0.4% interrupt, 92.5% idle
> Mem: 483M Active, 120M Inact, 222M Wired, 25M Cache, 112M Buf, 153M Free
> Swap: 2048M Total, 143M Used, 1905M Free, 6% Inuse

**首部的几行显示系统的几个信息，其中包括:**
+ Load averages:1分钟、5分钟和15分钟内运行的负载平均数
+ system:系统名和当前日期.
一般来说只要每个CPU的当前活动进程数不大于 3那么系统的性能就是良好的，如果每个CPU的任务数大于5，那么就表示这台机器的性能有严重问题
+ 最近一次更新时存在的进程总数，并分别列出run(运行)、sleep(睡眠)、idle（停止）和zomb(‘僵尸’)状态的进程数.
+ CPU state:用户占用时间的百分比、系统占用CPU时间的百分比、被nice命令改变优先级的任务占用的CPU时间百分比、以及CPU空闲时间的百分比。
（被nice命令改变优先级的任务仅指那些nice值为负的任务）。花费在被nice命令改变优先级的任务上的时间也将被计算在系统和用户时间内，因此整个时间加起来可能会超过百分之百.

**二．内存信息**
Memory: 610008K (24424K) real, 995344K (30304K) virtual, 12588K free Page# 1/4
Memory:关于内存使用情况的统计，包括实际（real）内存的活动值/总值，虚拟（virtual）内存的使用值/总值，剩余的内存。
DESCRIPTION OF MEMORY

> Mem: 9220K Active, 1032K Inact, 3284K Wired, 1MB Cache, 2M Buf, 1320K
> Free Swap: 91M Total, 79M Free, 13% Inuse, 80K In, 104 K Out

K: Kilobyte(K)
M: Megabyte(兆)
%: 1/100(百分比)
Active:活动页的数目
Inact: 非活动页的数目
Wired: 已经被写入页的数目, 包括缓存文件数据页码
Cache: 被用于 VM-level 磁盘缓冲的页的数目
Buf: 被用于 BIO-level 磁盘缓冲的页的数目
Free: 空闲页
Total: 总的可使用交换区
Free: 总共空闲的交换区
Inuse: 交换区的使用情况
In: pages paged in from swap devices (最近的时间间隔)
Out: pages paged out to swap devices (最近的时间间隔)

 **三．进程信息**
CPU PID USERNAME PRI NI SIZE RES STATE TIME %WCPU %CPU COMMAND
1 33 root 152 20 0K 0K run 153:43 1.18 1.18 vxfsd
0 1751 root 154 20 2500K 868K sleep 2084:19 0.52 0.52 ARMServer
0 1730 root 154 20 4500K 332K sleep 1664:55 0.44 0.44 acactmgr
列出系统里每一个处理器的信息,当信息在一个屏幕内无法显示时,会被分成多个屏幕显示,可以前面提到l,k和t命令查看
（1）CPU：处理器号（仅当多处理器系统时列出）
（2）PID：进程号
（3）USERNAME：用户名
（4）PRI:任务的优先级
（5）NICE：任务的nice值，一个具有较低值的进程在系统上将具有优先权。可以通过改变nice值提高某些进程速度，但是这实际上是一种交易，
因为那些nice值被升高的进程此时将运行得很慢。
（6）SIZE：任务的代码加上数据再加上栈空间的大小。
（7）RES：任务使用的物理内存的总数量。
（8）STATE：任务的状态
（9）TIME：自任务开始时使用的总CPU时间,单位为秒，如153:43，对应是153秒43毫秒
（10）%WCPU：进程的CPU利用率权重百分比
（11）%CPU：进程的原始的CPU利用率百分比，自上一次屏幕刷新以来任务占用CPU 时间的份额
（12）COMMAND：启动进程的命令名。如果名字太长而不能在一行显示时，它将被截短