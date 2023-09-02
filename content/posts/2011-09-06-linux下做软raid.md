---
title: Linux下做软RAID
author: admin
type: post
date: 2011-09-05T23:44:30+00:00
url: /archives/11308
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - 软RAID
 - mdadm
 - RAID

---
GUI:安装CentOS5.0过程中做软RAID:

CLI:Linux下做软raid: [http://docs.haohtml.com/download/linux/LINUX%c8%edRAID.pdf](http://docs.haohtml.com/download/linux/LINUX%c8%edRAID.pdf)

========  **mdadm使用详解**======================

**

**

**★mdadm简介**

我们可以使用man mdadm命令来查看mdadm的帮助信息：

[root@localhost mdadm-2.6.2]# man mdadm

**☆mdadm用法**
**基本语法**：

mdadm [mode]  [options]

**目前支持**：

LINEAR, RAID0(striping), RAID1(mirroring), RAID4, RAID5, RAID6, RAID10, MULTIPATH和FAULTY

**模式(7种)：**

 * Assemble：加入一个以前定义的阵列
 * Build：创建一个没有超级块的阵列
 * Create：创建一个新的阵列，每个设备具有超级块
 * Manage： 管理阵列(如添加和删除)
 * Misc：允许单独对阵列中的某个设备进行操作(如停止阵列)
 * Follow or Monitor:监控RAID的状态
 * Grow：改变RAID的容量或阵列中的设备数目

**选项：**
-A, –assemble：加入一个以前定义的阵列
-B, –build：创建一个没有超级块的阵列(Build a legacy array without superblocks.)
-C, –create：创建一个新的阵列
-F, –follow, –monitor：选择监控(Monitor)模式
-G, –grow：改变激活阵列的大小或形态
-I, –incremental：添加一个单独的设备到合适的阵列，并可能启动阵列
–auto-detect：请求内核启动任何自动检测到的阵列
-h, –help：帮助信息，用在以上选项后，则显示该选项信息
–help-options：显示更详细的帮助
-V, –version：打印mdadm的版本信息
-v, –verbose：显示细节
-b, –brief：较少的细节。用于 –detail 和 –examine 选项
-Q, –query：查看一个device，判断它为一个 md device 或是 一个 md 阵列的一部分
-D, –detail：打印一个或多个 md device 的详细信息
-E, –examine：打印 device 上的 md superblock 的内容
-c, –config= ：指定配置文件，缺省为 /etc/mdadm.conf
-s, –scan：扫描配置文件或 /proc/mdstat以搜寻丢失的信息。配置文件/etc/mdadm.conf

**★使用mdadm创建RAID5**

Create (mdadm –create)模式用来创建一个新的阵列。 在这里我们首先使用mdadm –create –help查看一下帮助：

> [root@localhost mdadm-2.6.2]# mdadm –create –help
> Usage:  mdadm –create device -chunk=X –level=Y –raid-devices=Z devices
> This usage will initialise a new md array, associate some
> devices with it, and activate the array.   In order to create an
> array with some devices missing, use the special word ‘missing’ in
> place of the relevant device name.
> Before devices are added, they are checked to see if they already contain
> raid superblocks or filesystems.  They are also checked to see if
> the variance in device size exceeds 1%.
> If any discrepancy is found, the user will be prompted for confirmation
> before the array is created.  The presence of a ‘–run’ can override this
> caution.
> If the –size option is given then only that many kilobytes of each
> device is used, no matter how big each device is.
> If no –size is given, the apparent size of the smallest drive given
> is used for raid level 1 and greater, and the full device is used for
> other levels.
> Options that are valid with –create (-C) are:
> –bitmap=          : Create a bitmap for the array with the given filename
> –chunk=      -c   : chunk size of kibibytes
> –rounding=        : rounding factor for linear array (==chunk size)
> –level=      -l   : raid level: 0,1,4,5,6,linear,multipath and synonyms
> –parity=     -p   : raid5/6 parity algorithm: {left,right}-{,a}symmetric
> –layout=          : same as –parity
> –raid-devices= -n : number of active devices in array
> –spare-devices= -x: number of spares (eXtras) devices in initial array
> –size=       -z   : Size (in K) of each drive in RAID1/4/5/6/10 – optional
> –force       -f   : Honour devices as listed on command line.  Don’t
> : insert a missing drive for RAID5.
> –run         -R   : insist of running the array even if not all
> : devices are present or some look odd.
> –readonly    -o   : start the array readonly – not supported yet.
> –name=       -N   : Textual name for array – max 32 characters
> –bitmap-chunk=    : bitmap chunksize in Kilobytes.
> –delay=      -d   : bitmap update delay in seconds.

接下来我们使用mdadm创建在/dev/md0上创建一个由sdb、sdc、sdd3块盘组成(另外1块盘sde为热备)的RAID5：

> [root@localhost mdadm-2.6.2]# mdadm –create –verbose /dev/md0 –level=raid5 –raid-devices=3 /dev/sdb /dev/sdc /dev/sdd –spare-devices=1 /dev/sde
> mdadm: layout defaults to left-symmetric
> mdadm: chunk size defaults to 64K
> mdadm: size set to 8388544K
> mdadm: array /dev/md0 started.

每个mdadm的选项都有一个缩写的形式，例如，上面我们创建RAID 5的命令可以使用下列的缩写形式：

> [root@localhost mdadm-2.6.2]# mdadm -Cv /dev/md0 -l5 -n3 /dev/sdb /dev/sdc /dev/sdd -x1 /dev/sde

二者的效果是相同的。

**★查看RAID状态**

接下来我们使用cat /proc/mdstat命令来查看一下RAID的状态，我们也可以利用watch命令来每隔一段时间刷新/proc/mdstat的输出。使用CTRL+C可以取消。

> [root@localhost mdadm-2.6.2]# watch -n 10 ‘cat /proc/mdstat’
> Every 10s: cat /proc/mdstat                             Thu May 24 11:53:46 2007
> Personalities : [raid5]
> read_ahead 1024 sectors
> md0 : active raid5 sdd[4] sde[3] sdc[1] sdb[0]
> 16777088 blocks level 5, 64k chunk, algorithm 2 \[3/2\] \[UU_\]
> [====>…………….]  recovery = 24.0% (2016364/8388544) finish=10.2min
> speed=10324K/sec
> unused devices:
> [root@localhost mdadm-2.6.2]#

接下来我们为阵列创建文件系统：

> [root@localhost mdadm-2.6.2]# mkfs.ext3 /dev/md0
> mke2fs 1.34 (25-Jul-2003)
> Filesystem label=
> OS type: Linux
> Block size=4096 (log=2)
> Fragment size=4096 (log=2)
> 2097152 inodes, 4194272 blocks
> 209713 blocks (5.00%) reserved for the super user
> First data block=0
> 128 block groups
> 32768 blocks per group, 32768 fragments per group
> 16384 inodes per group
> Superblock backups stored on blocks:
> 32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
> 4096000
> Writing inode tables: done
> Creating journal (8192 blocks): done
> Writing superblocks and filesystem accounting information: done
> This filesystem will be automatically checked every 34 mounts or
> 180 days, whichever comes first.  Use tune2fs -c or -i to override.
> You have new mail in /var/spool/mail/root

我们尝试向RAID中写入一个test2文件：

> [root@localhost eric4ever]# vi test2
> copy succeed!
> eric@tlf
> [url]http://eric4ever.googlepages.com/[/url]
> done!
> [root@localhost eric4ever]# ls
> LATEST.tgz  mdadm-2.6.2  test2
> [root@localhost eric4ever]# mount /dev/md0 /mnt/md0
> [root@localhost eric4ever]# df -lh
> Filesystem            Size  Used Avail Use% Mounted on
> /dev/sda1             2.9G  1.8G  1.1G  63% /
> /dev/sda3             4.6G   33M  4.3G   1% /opt
> none                  125M     0  125M   0% /dev/shm
> /dev/md0               16G   33M   15G   1% /mnt/md0
> [root@localhost eric4ever]# ls /mnt/md0
> lost+found
> [root@localhost eric4ever]# cp ./test2 /mnt/md0
> [root@localhost eric4ever]# ls /mnt/md0
> lost+found  test2
> [root@localhost eric4ever]# ls -lh /mnt/md0
> total 20K
> drwx——    2 root     root          16K May 24 11:55 lost+found
> -rw-r–r–    1 root     root           63 May 24 11:56 test2

使用mdadm –detail /dev/md0(或mdadm -D /dev/md0)命令以及cat /proc/mdstat命令可以查看RAID设备的状态：

> [root@localhost eric4ever]# mdadm -D /dev/md0  (或mdadm –detail /dev/md0)
> /dev/md0:
> Version : 00.90.00
> Creation Time : Thu May 24 13:45:35 2007
> Raid Level : raid5
> Array Size : 16777088 (16.00 GiB 17.18 GB)
> Used Dev Size : 8388544 (8.00 GiB 8.59 GB)
> Raid Devices : 3
> Total Devices : 5
> Preferred Minor : 0
> Persistence : Superblock is persistent
> Update Time : Thu May 24 13:45:36 2007
> State : active, degraded, recovering
> Active Devices : 2
> Working Devices : 4
> Failed Devices : 1
> Spare Devices : 2
> Layout : left-symmetric
> Chunk Size : 64K
> Rebuild Status : 16% complete
> UUID : 4b15050e:7d0c477d:98ed7d00:0f3c29e4
> Events : 0.2
> Number   Major   Minor   RaidDevice State
> 0       8       16        0      active sync   /dev/sdb
> 1       8       32        1      active sync   /dev/sdc
> 2       0        0        2      removed
> 3       8       64        3      spare   /dev/sde
> 4       8       48        4      spare   /dev/sdd

通过mdadm -D命令，我们可以查看RAID的版本、创建的时间、RAID级别、阵列容量、可用空间、设备数量、超级块、更新时间、各个设备的状态、RAID算法以及块大小等信息，通过上面的信息我们可以看到目前RAID正处于重建过程之中，进度为16%，其中/dev/sdb和/dev/sdc两块盘已经同步。使用watch命令每个30秒刷新一下查看的进度：

> [root@localhost eric4ever]# watch -n 30 ‘cat /proc/mdstat’
> Every 30s: cat /proc/mdstat                             Thu May 24 13:55:56 2007
> Personalities : [raid5]
> read_ahead 1024 sectors
> md0 : active raid5 sdd[4] sde[3] sdc[1] sdb[0]
> 16777088 blocks level 5, 64k chunk, algorithm 2 \[3/2\] \[UU_\]
> [==============>……]  recovery = 72.3% (6067444/8388544) finish=3.7min
> speed=10324K/sec
> unused devices:

当进度进行到100%时，显示如下：

> Every 30s: cat /proc/mdstat                             Thu May 24 14:00:57 2007
> Personalities : [raid5]
> read_ahead 1024 sectors
> md0 : active raid5 sdd[2] sde[3] sdc[1] sdb[0]
> 16777088 blocks level 5, 64k chunk, algorithm 2 \[3/3\] \[UUU\]
> unused devices:

这是我们再使用mdadm -D命令查看一下：

> [root@localhost eric4ever]# mdadm -D /dev/md0  (或mdadm –detail /dev/md0)
> /dev/md0:
> Version : 00.90.00
> Creation Time : Thu May 24 13:45:35 2007
> Raid Level : raid5
> Array Size : 16777088 (16.00 GiB 17.18 GB)
> Used Dev Size : 8388544 (8.00 GiB 8.59 GB)
> Raid Devices : 3
> Total Devices : 5
> Preferred Minor : 0
> Persistence : Superblock is persistent
> Update Time : Thu May 24 13:59:48 2007
> State : active
> Active Devices : 3
> Working Devices : 4
> Failed Devices : 1
> Spare Devices : 1
> Layout : left-symmetric
> Chunk Size : 64K
> UUID : 4b15050e:7d0c477d:98ed7d00:0f3c29e4
> Events : 0.3
> Number   Major   Minor   RaidDevice State
> 0       8       16        0      active sync   /dev/sdb
> 1       8       32        1      active sync   /dev/sdc
> 2       8       48        2      active sync   /dev/sdd
> 3       8       64        3      spare   /dev/sde
>
>

这时我们可以发现sdb、sdc、sdd三块盘均已经同步。
一般来说，一个新阵列被创建后我们最好创建一个/etc/mdadm.conf文件。没有该文件在激活阵列时我们就得指定更详细的信息，为方便，我们使用下列命令：

> [root@localhost eric4ever]# mdadm –detail –scan
> ARRAY /dev/md0 level=raid5 num-devices=3 spares=1 UUID=4b15050e:7d0c477d:98ed7d00:0f3c29e4
> [root@localhost eric4ever]# mdadm –detail –scan >> /etc/mdadm.conf
> [root@localhost eric4ever]# cat /etc/mdadm.conf
> ARRAY /dev/md0 level=raid5 num-devices=3 spares=1 UUID=4b15050e:7d0c477d:98ed7d00:0f3c29e4

**★启动停止RAID**

使用–stop或-S命令选项可以停止运行的阵列(**注意：** 停止前必须先umount)：

> [root@localhost eric4ever]# umount /mnt/md0
> [root@localhost eric4ever]# mdadm -S /dev/md0  (或mdadm –stop /dev/md0)
> mdadm: stopped /dev/md0

重新启动可以使用：

> [root@localhost eric4ever]# mdadm -As /dev/md0
> mdadm: /dev/md0 has been started with 3 drives and 1 spare.

**★模拟故障**

同raidtools一样，mdadm也可以软件模拟故障，命令选项为–fail或–set-faulty：

> [root@localhost eric4ever]# mdadm –set-faulty –help
> Usage: mdadm arraydevice options component devices…
> This usage is for managing the component devices within an array.
> The –manage option is not needed and is assumed if the first argument
> is a device name or a management option.
> The first device listed will be taken to be an md array device, and
> subsequent devices are (potential) components of that array.
> Options that are valid with management mode are:
> –add         -a   : hotadd subsequent devices to the array
> –remove      -r   : remove subsequent devices, which must not be active
> –fail        -f   : mark subsequent devices a faulty
> –set-faulty       : same as –fail
> –run         -R   : start a partially built array
> –stop        -S   : deactivate array, releasing all resources
> –readonly    -o   : mark array as readonly
> –readwrite   -w   : mark array as readwrite
> [root@localhost eric4ever]# mdadm –fail –help
> Usage: mdadm arraydevice options component devices…
> This usage is for managing the component devices within an array.
> The –manage option is not needed and is assumed if the first argument
> is a device name or a management option.
> The first device listed will be taken to be an md array device, and
> subsequent devices are (potential) components of that array.
> Options that are valid with management mode are:
> –add         -a   : hotadd subsequent devices to the array
> –remove      -r   : remove subsequent devices, which must not be active
> –fail        -f   : mark subsequent devices a faulty
> –set-faulty       : same as –fail
> –run         -R   : start a partially built array
> –stop        -S   : deactivate array, releasing all resources
> –readonly    -o   : mark array as readonly
> –readwrite   -w   : mark array as readwrite

接下来我们模拟/dev/sdb故障：

> [root@localhost eric4ever]# mdadm –manage –set-faulty /dev/md0 /dev/sdb
> mdadm: set /dev/sdb faulty in /dev/md0

查看一下系统日志，如果你配置了冗余磁盘，可能会显示如下信息：

> kernel: raid5: Disk failure on sdb, disabling device.
> kernel: md0: resyncing spare disk sde to replace failed disk

检查/proc/mdstat，如果配置的冗余磁盘可用，阵列可能已经开始重建。
首先我们使用mdadm –detail /dev/md0命令来查看一下RAID的状态：

> [root@localhost eric4ever]# mdadm –detail /dev/md0
> /dev/md0:
> Version : 00.90.00
> Creation Time : Thu May 24 13:45:35 2007
> Raid Level : raid5
> Array Size : 16777088 (16.00 GiB 17.18 GB)
> Used Dev Size : 8388544 (8.00 GiB 8.59 GB)
> Raid Devices : 3
> Total Devices : 5
> Preferred Minor : 0
> Persistence : Superblock is persistent
> Update Time : Thu May 24 14:07:55 2007
> State : active, degraded, recovering
> Active Devices : 2
> Working Devices : 3
> Failed Devices : 2
> Spare Devices : 1
> Layout : left-symmetric
> Chunk Size : 64K
> Rebuild Status : 3% complete
> UUID : 4b15050e:7d0c477d:98ed7d00:0f3c29e4
> Events : 0.6
> Number   Major   Minor   RaidDevice State
> 0       8       16        0      faulty spare   /dev/sdb
> 1       8       32        1      active sync   /dev/sdc
> 2       8       48        2      active sync   /dev/sdd
> 3       8       64        3      spare rebuilding   /dev/sde

查看/proc/mdstat：

> [root@localhost eric4ever]# cat /proc/mdstat
> Personalities : [raid5]
> read_ahead 1024 sectors
> md0 : active raid5 sdb[4] sde[3] sdd[2] sdc[1]
> 16777088 blocks level 5, 64k chunk, algorithm 2 \[3/2\] \[_UU\]
> [==>………………]  recovery = 10.2% (858824/8388544) finish=12.4min speed=10076K/sec
> unused devices:

再查看一下RAID状态：

> [root@localhost eric4ever]# mdadm –detail /dev/md0
> /dev/md0:
> Version : 00.90.00
> Creation Time : Thu May 24 13:45:35 2007
> Raid Level : raid5
> Array Size : 16777088 (16.00 GiB 17.18 GB)
> Used Dev Size : 8388544 (8.00 GiB 8.59 GB)
> Raid Devices : 3
> Total Devices : 5
> Preferred Minor : 0
> Persistence : Superblock is persistent
> Update Time : Thu May 24 14:08:27 2007
> State : active, degraded, recovering
> Active Devices : 2
> Working Devices : 4
> Failed Devices : 1
> Spare Devices : 2
> Layout : left-symmetric
> Chunk Size : 64K
> Rebuild Status : 11% complete
> UUID : 4b15050e:7d0c477d:98ed7d00:0f3c29e4
> Events : 0.8
> Number   Major   Minor   RaidDevice State
> 0       0        0        0      removed
> 1       8       32        1      active sync   /dev/sdc
> 2       8       48        2      active sync   /dev/sdd
> 3       8       64        3      spare   /dev/sde
> 4       8       16        4      spare   /dev/sdb

已经完成到11%了。查看一下日志消息：

> [root@localhost eric4ever]# tail /var/log/messages
> May 24 14:08:27 localhost kernel:  — rd:3 wd:2 fd:1
> May 24 14:08:27 localhost kernel:  disk 0, s:0, o:0, n:0 rd:0 us:0 dev:[dev 00:00]
> May 24 14:08:27 localhost kernel:  disk 1, s:0, o:1, n:1 rd:1 us:1 dev:sdc
> May 24 14:08:27 localhost kernel:  disk 2, s:0, o:1, n:2 rd:2 us:1 dev:sdd
> May 24 14:08:27 localhost kernel: RAID5 conf printout:
> May 24 14:08:27 localhost kernel:  — rd:3 wd:2 fd:1
> May 24 14:08:27 localhost kernel:  disk 0, s:0, o:0, n:0 rd:0 us:0 dev:[dev 00:00]
> May 24 14:08:27 localhost kernel:  disk 1, s:0, o:1, n:1 rd:1 us:1 dev:sdc
> May 24 14:08:27 localhost kernel:  disk 2, s:0, o:1, n:2 rd:2 us:1 dev:sdd
> May 24 14:08:27 localhost kernel: md: cannot remove active disk sde from md0 …

使用mdadm -E命令查看一下/dev/sdb的情况：

> [root@localhost eric4ever]# mdadm -E /dev/sdb
> /dev/sdb:
> Magic : a92b4efc
> Version : 00.90.00
> UUID : 4b15050e:7d0c477d:98ed7d00:0f3c29e4
> Creation Time : Thu May 24 13:45:35 2007
> Raid Level : raid5
> Used Dev Size : 8388544 (8.00 GiB 8.59 GB)
> Array Size : 16777088 (16.00 GiB 17.18 GB)
> Raid Devices : 3
> Total Devices : 5
> Preferred Minor : 0
> Update Time : Thu May 24 14:08:27 2007
> State : active
> Active Devices : 2
> Working Devices : 4
> Failed Devices : 1
> Spare Devices : 2
> Checksum : a6a19662 – correct
> Events : 0.8
> Layout : left-symmetric
> Chunk Size : 64K
> Number   Major   Minor   RaidDevice State
> this     4       8       16        4      spare   /dev/sdb
> 0     0       0        0        0      faulty removed
> 1     1       8       32        1      active sync   /dev/sdc
> 2     2       8       48        2      active sync   /dev/sdd
> 3     3       8       64        3      spare   /dev/sde
> 4     4       8       16        4      spare   /dev/sdb

自动修复完成后，我们再查看一下RAID的状态：

> [root@localhost eric4ever]# mdadm –detail /dev/md0
> /dev/md0:
> Version : 00.90.00
> Creation Time : Thu May 24 13:45:35 2007
> Raid Level : raid5
> Array Size : 16777088 (16.00 GiB 17.18 GB)
> Used Dev Size : 8388544 (8.00 GiB 8.59 GB)
> Raid Devices : 3
> Total Devices : 5
> Preferred Minor : 0
> Persistence : Superblock is persistent
> Update Time : Thu May 24 14:21:54 2007
> State : active
> Active Devices : 3
> Working Devices : 4
> Failed Devices : 1
> Spare Devices : 1
> Layout : left-symmetric
> Chunk Size : 64K
> UUID : 4b15050e:7d0c477d:98ed7d00:0f3c29e4
> Events : 0.9
> Number   Major   Minor   RaidDevice State
> 0       8       64        0      active sync   /dev/sde
> 1       8       32        1      active sync   /dev/sdc
> 2       8       48        2      active sync   /dev/sdd
> 4       8       16        4      spare   /dev/sdb
> [root@localhost eric4ever]# cat /proc/mdstat
> Personalities : [raid5]
> read_ahead 1024 sectors
> md0 : active raid5 sdb[4] sde[0] sdd[2] sdc[1]
> 16777088 blocks level 5, 64k chunk, algorithm 2 \[3/3\] \[UUU\]
> unused devices:

我们可以看到/dev/sde已经替换了/dev/sdb。看看系统的日志消息：

> [root@localhost eric4ever]# tail /var/log/messages
> May 24 14:21:54 localhost kernel:  — rd:3 wd:3 fd:0
> May 24 14:21:54 localhost kernel:  disk 0, s:0, o:1, n:0 rd:0 us:1 dev:sde
> May 24 14:21:54 localhost kernel:  disk 1, s:0, o:1, n:1 rd:1 us:1 dev:sdc
> May 24 14:21:54 localhost kernel:  disk 2, s:0, o:1, n:2 rd:2 us:1 dev:sdd
> May 24 14:21:54 localhost kernel: md: updating md0 RAID superblock on device
> May 24 14:21:54 localhost kernel: md: sdb [events: 00000009]<6>(write) sdb’s sb offset: 8388544
> May 24 14:21:54 localhost kernel: md: sde [events: 00000009]<6>(write) sde’s sb offset: 8388544
> May 24 14:21:54 localhost kernel: md: sdd [events: 00000009]<6>(write) sdd’s sb offset: 8388544
> May 24 14:21:54 localhost kernel: md: sdc [events: 00000009]<6>(write) sdc’s sb offset: 8388544
> May 24 14:21:54 localhost kernel: md: recovery thread got woken up …
>
> recovery thread got woken up …

这时我们可以从/dev/md0中移除/dev/sdb设备：

> [root@localhost eric4ever]# mdadm /dev/md0 -r /dev/sdb
> mdadm: hot removed /dev/sdb

类似地，我们可以使用下列命令向/dev/md0中添加一个设备：

> [root@localhost eric4ever]# mdadm /dev/md0 –add /dev/sdf

**★监控RAID**

mdadm的监控模式提供一些实用的功能，你可以使用下列命令来监控/dev/md0，delay参数意味着检测的时间间隔，这样紧急事件和严重的错误会及时发送给系统管理员：

> [root@localhost eric4ever]# mdadm –monitor –mail=eric4ever@localhost –delay=300 /dev/md0

当使用监控模式时，mdadm不会退出，你可以使用下列命令：

> [root@localhost eric4ever]# nohup mdadm –monitor –mail=eric4ever@localhost –delay=300 /dev/md0 &
> [1] 3113
> [root@localhost eric4ever]# nohup: appending output to `nohup.out’

此教程相对来说只是简介了如何做软raid了,对于添加硬盘扩展空间和删除一块硬盘没有做详细的介绍,这时可以参考: [http://docs.haohtml.com/download/linux/LINUX%c8%edRAID.pdf](http://docs.haohtml.com/download/linux/LINUX%c8%edRAID.pdf) 教程,这里介绍的比较的详细的.

如果想创建lvm的话,可以继续教程:,从教程的第(三)开始即可.因为默认已经有了VolGroup**卷组**了.无需再创建了.