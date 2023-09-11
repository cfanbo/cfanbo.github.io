---
title: FreeBSD/Linux检测硬盘坏道
author: admin
type: post
date: 2011-10-20T05:11:35+00:00
url: /archives/11806
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux

---
**Linux检测硬盘坏道**

**badblocks**

功能说明：检查磁盘装置中损坏的区块。

语法：badblocks \[-svw\]\[-b \]\[-o \]\[磁盘装置\]\[磁盘区块数\]\[启始区块\]

补充说明：执行指令时须指定所要检查的磁盘装置，及此装置的磁盘区块数。

**参数：**

-b 指定磁盘的区块大小，单位为字节。

-o 将检查的结果写入指定的输出文件。

-s 在检查时显示进度。

-v 执行时显示详细的信息。

-w 在检查时，执行写入测试。

[磁盘装置] 指定要检查的磁盘装置。

[磁盘区块数] 指定磁盘装置的区块总数。

[启始区块] 指定要从哪个区块开始检查。

badblocks 检测磁盘坏块

1)$badblocks -s //显示进度 -v //显示执行详细情况 /dev/sda1

2)读写方式检测 未挂载的磁盘设备或分区

$badblocks -s //显示进度 -w //以写去检测 -v //显示执行详细情况 /dev/sda2

=========================**FreeBSD检测硬盘坏道**

利用硬盘的S.M.A.R.T.功能来做。

> cd /usr/ports/sysutils/smartmontools
> make install clean

快速检查硬盘是否有问题

> smartctl -a /dev/ad0

表面测试

> smartctl -t long /dev/ad0

好像还有一个badtrk工具.

=========================================================================

**smartctl详解:**

监测你的硬盘 – 提前预报系统SMART

前言：

大家心理最怕的不是安装某个系统，而是辛辛苦苦安装之后，忽然有一天硬盘坏了，又没有备份(DAT，DLT之类磁带机贵得吓死人)。怎么样才能知道你的硬盘能否过新年呢?(硬盘状态如何?) 特别是如果能够提前预报，告诉大家硬盘快顶不住了，那该多好。

解决办法：

SMART

SMART(SFF-8035i) 是硬盘生产商们建立的一个工业标准，这个标准就是在硬盘上保存一个跟执行情况，可靠程度，读找错误率等属性的表格。所有属性都有一个1byte(大

小范围1-253)的标准化值，还包含另一个1byte的关键阶段值，如果属性表格内某个数据接近小于或达到关键阶段值，那么你的硬盘就快跟你永别了，至少也是超过它的设计使用极限了- 该做备份和最坏的打算了。

SFF-8035i工业标准经过ATA-3，ATA-4到了ATA-5，加入了一个错误信息文件(errorlog) 和一系列硬盘自测SMART命令。SMART适应与IDE和SCSI硬盘。

我用FreeBSD 5.2和Debian做了实验，都不错，OpenBSD下面可以直接用atactl，大家看看man atactl，或是下面的帖子。其它linux系统没问题，可以看文章最后给出的官方网站去查询一下你的系统。

1。安装 smartmontools

FreeBSD:

> #/usr/ports/sysutils/smartmontools
>
> #make install clean
>
> #cp /usr/local/etc/rc.d/smartd.sh.sample /usr/local/etc/rc.d/smartd.sh
>
> #cp /usr/local/etc/smartd.conf.sample /usr/local/etc/smartd.conf
>
> #chmod 555 /usr/local/etc/rc.d/smartd.sh

Debian:

> 　　apt-get install smartmontool*

/etc/smartd.conf配置文件:

FreeBSD设置文件/usr/local/etc/smartd.conf

Debian设置文件 /etc/smartd.conf

注意：

千万不要忘了改写设置文件!!!!

FreeBSD下第一张IDE硬盘是ad0，SCSI硬盘是da0

Debian下第一张IDE硬盘是/dev/hda，SCSI硬盘是/dev/sda

下面我用FreeBSD做例子，我的硬盘是IDE，如果你的是SCSI，你就去官方网站

启动监护程序：

> /usr/local/etc/rc.d/smartd.sh start

首先让我们看一下你的硬盘是否支持SMART：

bash-2.05b# smartctl -i /dev/ad0

smartctl version 5.26 Copyright (C) 2002-3 Bruce Allen

Home page is http://smartmontools.sourceforge.net/

=== START OF INFORMATION SECTION ===

Device Model: IBM-DJSA-220

Serial Number: 44K443Z0103

Firmware Version: JS4OAC3A

Device is: Not in smartctl database [for details use: -P showall]

ATA Version is: 5

ATA Standard is: ATA/ATAPI-5 T13 1321D revision 1

Local Time is: Mon Dec 22 21:04:38 2003 CET

SMART support is: Available – device has SMART capability.

SMART support is: enable

The SMART RETURN STATUS return value (smartmontools -H option/Directive)

can not be retrieved with this version of ATAng, please do not rely on this value

看看我的盘健康测试，如果你的self-assessment test result是FAILING，那就是说

它要完蛋了，马上备份!!!

bash-2.05b# smartctl -Hc /dev/ad0

smartctl version 5.26 Copyright (C) 2002-3 Bruce Allen

Home page is http://smartmontools.sourceforge.net/

The SMART RETURN STATUS return value (smartmontools -H option/Directive)

can not be retrieved with this version of ATAng, please do not rely on

this value

=== START OF READ SMART DATA SECTION ===

SMART overall-health self-assessment test result: PASSED

General SMART Values:

Offline data collection status: (0x00) Offline data collection activity

was

never started.

Auto Offline Data Collection: Disabled.

Self-test execution status: ( 0) The previous self-test routine completed

without error or no self-test has

ever

been run.

Total time to complete Offline

data collection: ( 650) seconds.

Offline data collection

capabilities: (0x1b) SMART execute Offline immediate.

Auto Offline data collection on/off

support.

Suspend Offline collection upon

new

command.

Offline surface scan supported.

Self-test supported.

No Conveyance Self-test supported.

No Selective Self-test supported.

SMART capabilities: (0x0003) Saves SMART data before entering

power-saving mode.

Supports SMART auto save timer.

Error logging capability: (0x01) Error logging supported.

No General Purpose Logging support.

Short self-test routine

recommended polling time: ( 2) minutes.

Extended self-test routine

recommended polling time: ( 29) minutes.

下面表格给出的属性信息根据你的硬盘厂商不同而不同，最 重要的是明白每个纵行的意义：如果有一个标准值(VALUE)小于或等於关键值(THRESH)时，WHEN\_FAILED 行会给出信息，我的WHEN\_FAILED纵行是空行，说明没事儿。如果WHEN_FAILED报错，硬盘有问题了。。。。WORST 是标准值(VALUE)的最小值。

bash-2.05b# smartctl -A /dev/ad0

smartctl version 5.26 Copyright (C) 2002-3 Bruce Allen

Home page is http://smartmontools.sourceforge.net/

The SMART RETURN STATUS return value (smartmontools -H option/Directive)

can not be retrieved with this version of ATAng, please do not rely on

this value

=== START OF READ SMART DATA SECTION ===

SMART Attributes Data Structure revision number: 16

Vendor Specific SMART Attributes with Thresholds:

ID# ATTRIBUTE_NAME FLAG VALUE WORST THRESH TYPE UPDATED

WHEN\_FAILED RAW\_VALUE

1 Raw\_Read\_Error_Rate 0x000b 100 100 062 Pre-fail Always

– 0

2 Throughput_Performance 0x0005 100 100 040 Pre-fail Offline

– 0

3 Spin\_Up\_Time 0x0007 113 113 033 Pre-fail Always

– 1

4 Start\_Stop\_Count 0x0012 100 100 000 Old_age Always

– 985

5 Reallocated\_Sector\_Ct 0x0033 100 100 005 Pre-fail Always

– 0

7 Seek\_Error\_Rate 0x000b 100 100 067 Pre-fail Always

– 0

8 Seek\_Time\_Performance 0x0005 100 100 040 Pre-fail Offline

– 0

9 Power\_On\_Hours 0x0012 097 097 000 Old_age Always

– 1642

10 Spin\_Retry\_Count 0x0013 100 100 060 Pre-fail Always

– 0

12 Power\_Cycle\_Count 0x0032 100 100 000 Old_age Always

– 914

191 G-Sense\_Error\_Rate 0x000a 100 100 000 Old_age Always

– 0

192 Power-Off\_Retract\_Count 0x0032 100 100 000 Old_age Always

– 8

193 Load\_Cycle\_Count 0x0012 096 096 050 Old_age Always

– 45262

196 Reallocated\_Event\_Count 0x0032 100 100 000 Old_age Always

– 17

197 Current\_Pending\_Sector 0x0022 100 100 000 Old_age Always

– 1

198 Offline\_Uncorrectable 0x0008 100 100 000 Old\_age Offline

– 0

199 UDMA\_CRC\_Error\_Count 0x000a 200 200 000 Old\_age Always

– 0

下面命令给出硬盘历史错误信息(error log),因为篇幅关系我就不给出了。

> smartctl -l error /dev/ad0

下面命令给出硬盘自测

> smartctl -l selftest /dev/ad0

终止硬盘自测。

> smartctl -X /dev/ad0

建议：改写设置文件smartd.conf，有一个“-m”的选项非常有用，它可以把信息用mail发给你。

**编辑后记：**

SMART 可以提醒你，但不能帮你做备份。要真正的让SMART为你服务，应该好好读写smartd & smartd.conf 的注释, 让其后台程序在一定情况下提醒你(mail)有些关键值达到了危险区域, 以上给出的几个命令是在你开始感到情况不妙的时候进行的手工测试。本文参考了英文杂志“Linux Journal January 2004″ – Monitor drive health with SMART, 作者是Bruce Allen物理教授。我是因为文章写的比自己的笔记好百倍，所以决定参考一些原文的例子和顺序。