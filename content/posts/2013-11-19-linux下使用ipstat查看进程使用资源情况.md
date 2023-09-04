---
title: linux下使用iostat和pidstat查看进程使用资源情况
author: admin
type: post
date: 2013-11-19T03:50:20+00:00
url: /archives/14760
categories:
 - 服务器
tags:
 - iostat
 - mpstat
 - pidstat

---

**引言**

在查看系统资源使用情况时，很多工具为我们提供了从设备角度查看的方法。例如使用 [iostat](http://www.cnblogs.com/bangerlee/articles/2547161.html) 查看磁盘io统计信息：


```
linux:~ # iostat -d 3
Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
sda               1.67         0.00        40.00                  120
```

以上显示的是从sda的角度统计的结果。当我们需要从进程的角度，查看每个进程使用系统资源的情况，有什么方法吗？


使用pidstat工具可以获取每个进程使用cpu、内存和磁盘等系统资源的统计信息，pidstat由sysstat rpm包提供，可在suse11使用。下面我们来看pidstat的具体用法。


**默认输出**

执行pidstat，将输出系统启动后所有活动进程的cpu统计信息：


```
linux:~ # pidstat
Linux 2.6.32.12-0.7-default (linux)             06/18/12        _x86_64_

11:37:19          PID    %usr %system  %guest    %CPU   CPU  Command
……
11:37:19        11452    0.00    0.00    0.00    0.00     2  bash
11:37:19        11509    0.00    0.00    0.00    0.00     3  dd
```

以上输出，除最开头一行显示内核版本、主机名、日期和cpu架构外，主要列含义如下：

- **11:37:19**: pidstat获取信息时间点

- **PID**: 进程pid

- **%usr**: 进程在用户态运行所占cpu时间比率

- **%system**: 进程在内核态运行所占cpu时间比率

- **%CPU**: 进程运行所占cpu时间比率

- **CPU**: 指示进程在哪个核运行

- **Command**: 拉起进程对应的命令


执行pidstat默认输出信息为系统启动后到执行时间点的统计信息，因而即使当前某进程的cpu占用率很高，输出中的值有可能仍为0。


**指定采样周期和采样次数**

像 [sar](http://www.cnblogs.com/bangerlee/articles/2545747.html)、iostat等命令一样，也可以给pidstat命令指定采样周期和采样次数，命令形式为”pidstat [option] interval [count]”，以下pidstat输出以2秒为采样周期，输出2次cpu使用统计信息：


```
linux:~ # pidstat 2 2
Linux 2.6.32.12-0.7-default (linux)             06/18/12        _x86_64_

14:40:39          PID    %usr %system  %guest    %CPU   CPU  Command
14:40:41         9567    0.50    1.49    0.00    1.98     2  atop
14:40:41        12405    0.00    0.50    0.00    0.50     6  pidstat

14:40:41          PID    %usr %system  %guest    %CPU   CPU  Command
14:40:43         7830    0.50    0.50    0.00    1.00     7  runHpiAlarm
14:40:43        12405    0.00    1.00    0.00    1.00     6  pidstat
```

若不指定统计次数count，则pidstat将一直输出统计信息。


**cpu使用情况统计(-u)**

使用-u选项，pidstat将显示各活动进程的cpu使用统计，执行”pidstat -u”与单独执行”pidstat”的效果一样。


**内存使用情况统计(-r)**

使用-r选项，pidstat将显示各活动进程的内存使用统计：


```
linux:~ # pidstat -r -p 13084 1
Linux 2.6.32.12-0.7-default (linux)             06/18/12        _x86_64_

15:08:18          PID  minflt/s  majflt/s     VSZ    RSS   %MEM  Command
15:08:19        13084 133835.00      0.00 15720284 15716896  96.26  mmmm
15:08:20        13084  35807.00      0.00 15863504 15849756  97.07  mmmm
15:08:21        13084  19273.87      0.00 15949040 15792944  96.72  mmmm
```

以上各列输出的含义如下：


- **minflt/s**: 每秒次缺页错误次数(minor page faults)，次缺页错误次数意即虚拟内存地址映射成物理内存地址产生的page fault次数

- **majflt/s**: 每秒主缺页错误次数(major page faults)，当虚拟内存地址映射成物理内存地址时，相应的page在swap中，这样的page fault为major page fault，一般在内存使用紧张时产生

- **VSZ**: 该进程使用的虚拟内存(以kB为单位)

- **RSS**: 该进程使用的物理内存(以kB为单位)

- **%MEM**: 该进程使用内存的百分比

- **Command**: 拉起进程对应的命令


**IO情况统计(-d)**

使用-d选项，我们可以查看进程IO的统计信息：


```
linux:~ # pidstat -d 1 2
Linux 2.6.32.12-0.7-default (linux)             06/18/12        _x86_64_

17:11:36          PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
17:11:37        14579 124988.24      0.00      0.00  dd

17:11:37          PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
17:11:38        14579 105441.58      0.00      0.00  dd
```

以上主要输出的含义如下：


- **kB_rd/s**: 每秒进程从磁盘读取的数据量(以kB为单位)

- **kB_wr/s**: 每秒进程向磁盘写的数据量(以kB为单位)

- **Command**: 拉起进程对应的命令


**针对特定进程统计(-p)**

使用-p选项，我们可以查看特定进程的系统资源使用情况：


```
linux:~ # pidstat -r -p 1 1
Linux 2.6.32.12-0.7-default (linux)             06/18/12        _x86_64_

18:26:17          PID  minflt/s  majflt/s     VSZ    RSS   %MEM  Command
18:26:18            1      0.00      0.00   10380    640   0.00  init
18:26:19            1      0.00      0.00   10380    640   0.00  init
……
```

以上pidstat命令以1秒为采样时间间隔，查看init进程的内存使用情况。


**pidstat常用命令**

使用pidstat进行问题定位时，以下命令常被用到：


**pidstat -u 1**

**pidstat -r 1**

**pidstat -d 1**

以上命令以1秒为信息采集周期，分别获取cpu、内存和磁盘IO的统计信息。


转自： [http://www.cnblogs.com/bangerlee/articles/2555307.html](http://www.cnblogs.com/bangerlee/articles/2555307.html)

**判断I/O瓶颈**

mpstat命令


命令：mpstat -P ALL 1 1000


结果显示：


[![mpstat](https://blogstatic.haohtml.com//uploads/2023/09/mpstat.png)](http://blog.haohtml.com/wp-content/uploads/2013/11/mpstat.png)

注意一下这里面的%iowait列，CPU等待I/O操作所花费的时间。这个值持续很高通常可能是I/O瓶颈所导致的。


通过这个参数可以比较直观的看出当前的I/O操作是否存在瓶颈。