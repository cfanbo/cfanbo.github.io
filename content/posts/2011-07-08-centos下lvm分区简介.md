---
title: CentOS下lvm分区简介
author: admin
type: post
date: 2011-07-08T05:33:58+00:00
url: /archives/10298
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - lvm

---
LVM 是逻辑盘卷管理器（ Logical Volume Manager ）的简称，是一种分区管理机制。 LVM 是建立在硬盘 和分区 之上的一个逻辑层，为文件系统屏蔽下层磁盘分区布局，从而提高磁盘分区管理的灵活性。

要配置LVM，可以按以下步骤进行：

1. 创建和初始化物理卷（Physical Volume），通过pvcreate建立pv，即pv阶段；

2. 添加物理卷到卷组（Volume Group），使用vgcreate加入多个pv成为vg，即vg阶段；

3. 在卷组上创建逻辑卷（logical volume），使用lvcreate划分vg,成为一个或多个lv,即lv阶段；

[![](http://blog.haohtml.com/wp-content/uploads/2011/07/lvm-pic.bmp)](http://blog.haohtml.com/wp-content/uploads/2011/07/lvm-pic.bmp) 上图参考： [http://www.haohtml.com/server/unix/46733.html](http://www.haohtml.com/server/unix/46733.html)

具体思路是：将若干个磁盘分区连接为一个整块的卷组（ Vloume group ），管理员可以在卷组上随意创建逻辑卷（ logical volumes ），并进一步在逻辑卷上创建文件系统。

**物理卷（ Physical Volume ， PV ）**

PV 在 LVM 系统中处于最底层,PV 一般是整个硬盘、或硬盘上一个可用分区

**卷组（ Volume Group ， VG ）**
建立在 PV 之上，可以由多个 PV 组成一个 VG ，也可以是单个.VG 创建之后，可以动态地添加 PV 到 VG 中，在 VG 上一个创建多个 LVM 分区（逻辑卷）

一个 LVM 系统中可以包含多个 VG（注释：在这 LVM 系统中你可以把 VG 理解为实际的物理硬盘）


**逻辑卷（ Logical Volume ， LV ）**
LV 建立在 LV 之上（类似于 Windows 下的对物理硬盘分区，或者是 Linux 下分出 /boot 、 / 、 Swap 、 /usr 、/var 、 /tmp 、 /home 等区域）

LV 创建后，大小可以改变.在 LV 上可以建立文件系统用于不同的分区，如 /usr 、 /home

**1.1    物理区域（ Physical Extent ， PE ）**

每个 PV 又是由 PE 组成                     ←这类似于 Windows 下的分区格式化中的“簇”概念

PE 的大小默认为 4MB

PE 的大小一旦确定不能改变，同一 LG 中的所有 PV 的 PE 的大小又要一致

1.1.1  逻辑区域（ Logical Extent ， LE ）

LE 一一对映 PE ，所以 PE 单元为多大，映射到 LE 单元就是多大

当 LVM 执行完成之后， LV 及 VG 的相关元数据保留在位于 PV 起始处的卷组描述符区域 Volume Group Descriptor Area         ←这和 Windows 下的分区表是一样的

/boot 分区不能位于 LG 中，因为引导装载程序无法从逻辑卷中读取。必须为 /boot 单独分配一个主分区

CentOS 系统分区规划：

> /boot                 [ext3 ]                 →单独的 /boot 分区                  [256MB]

将剩余的可利用空间创建为一个 PV

将此 PV 加入到名为 VolGroup00 的 VG 中

在此 VG 中分别创建名为 LogVol100 和 LogVol101 和 LogVol102 和 LogVol103 和 LogVol104 和 LogVol105 和LogVol106 的 LV

> LV root:            [ext3 ]                 LogVol100         → /                      [20GB]
>
> LV swap:          [ext3 ]                 LogVol101         → swap             [ 物理内存的两倍 ]
>
> LV usr:               [ext3 ]                 LogVol102         → /usr                [20GB]
>
> LV tmp:              [ext3 ]                 LogVol103         → /tmp              [10GB]
>
> LV var:               [ext3 ]                 LogVol104         → /var                [10GB]
>
> LV home:           [ext3 ]                 LogVol105         → /home           [20GB]

（注意：如 LogVol100 这样的名称是系统默认分配的，最好别去重命名）

如果以上全部创建完成，并且安装完了系统，可以在 /dev/ 下查看磁盘分区信息，你将会看到类似如下：

> /dev/VolGropu00/LogVol100
>
> /dev/VolGropu00/LogVol101
>
> /dev/VolGropu00/LogVol102
>
> /dev/VolGropu00/LogVol103
>
> /dev/VolGropu00/LogVol104
>
> /dev/VolGropu00/LogVol105

参考：创建Raid阵列和lvm逻辑卷组： [http://docs.haohtml.com/download/linux/%b4%b4%bd%a8Raid%d5%f3%c1%d0%ba%cdlvm%c2%df%bc%ad%be%ed%d7%e9.pdf](http://docs.haohtml.com/download/linux/%b4%b4%bd%a8Raid%d5%f3%c1%d0%ba%cdlvm%c2%df%bc%ad%be%ed%d7%e9.pdf)