---
title: linux的mount（挂载）命令详解
author: admin
type: post
date: 2011-05-28T12:09:13+00:00
url: /archives/9583
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - mount

---
**点评：**linux下挂载（mount）光盘镜像文件、移动硬盘、U盘、Windows和NFS网络共享 linux是一个优秀的开放源码的操作系统，可以运行在大到巨型小到掌上型各类计算机系统上，随着 linux系统的日渐成熟和稳定以及它开放源代码特有的优越性，linux在全世界得到了越来越广泛的linux下挂载（mount）光盘镜像文件、移动硬盘、U盘、Windows和NFS网络共享.
linux是一个优秀的开放源码的操作系统，可以运行在大到巨型小到掌上型各类计算机系统上，随着 linux系统的日渐成熟和稳定以及它开放源代码特有的优越性，linux在全世界得到了越来越广泛的应用。现在许多企业的计算机系统都是由UNIX系 统、Linux系统和Windows系统组成的混合系统，不同系统之间经常需要进行数据交换。下面我根据自己的实际工作经验介绍一下如何在linux系统 下挂接(mount)光盘镜像文件、移动硬盘、U盘以及Windows网络共享和UNIX NFS网络共享。

**挂接命令(mount)**

首先，介绍一下挂接(mount)命令的使用方法，mount命令参数非常多，这里主要讲一下今天我们要用到的。

**命令格式：**

> mount \[-t vfstype\] \[-o options\] device dir

**其中：**

1.-t vfstype 指定文件系统的类型，通常不必指定。mount 会自动选择正确的类型。常用类型有：

光盘或光盘镜像：iso9660

DOS fat16文件系统：msdos

Windows 9x fat32文件系统：vfat

Windows NT ntfs文件系统：ntfs

Mount Windows文件网络共享：smbfs

UNIX(LINUX) 文件网络共享：nfs

2.-o options 主要用来描述设备或档案的挂接方式。常用的参数有：

loop：用来把一个文件当成硬盘分区挂接上系统

ro：采用只读方式挂接设备

rw：采用读写方式挂接设备

iocharset：指定访问文件系统所用字符集

3.device 要挂接(mount)的设备。4.dir 设备在系统上的挂接点(mount point)。



**一.挂接光盘镜像文件**

由于近年来磁盘技术的巨大进步，新的电脑系统都配备了大容量的磁盘系统，在Windows下许多人都习惯把软件和资料做成光盘镜像文件通过虚拟 光驱来使用。这样做有许多好处：一、减轻了光驱的磨损;二、现在硬盘容量巨大存放几十个光盘镜像文件不成问题，随用随调十分方便;三、硬盘的读取速度要远 远高于光盘的读取速度，CPU占用率大大降低。其实linux系统下制作和使用光盘镜像比Windows系统更方便，不必借用任何第三方软件包。

1、从光盘制作光盘镜像文件。将光盘放入光驱，执行下面的命令。

> #cp /dev/cdrom /home/sunky/mydisk.iso 或
>
> #dd if=/dev/cdrom of=/home/sunky/mydisk.iso

注：执行上面的任何一条命令都可将当前光驱里的光盘制作成光盘镜像文件/home/sunky/mydisk.iso

2、将文件和目录制作成光盘镜像文件，执行下面的命令。

> #mkisofs -r -J -V mydisk -o /home/sunky/mydisk.iso /home/sunky/mydir

注：这条命令将/home/sunky/mydir目录下所有的目录和文件制作成光盘镜像文件/home/sunky/mydisk.iso，光盘卷标为：mydisk

3、光盘镜像文件的挂接(mount)

> #mkdir /mnt/cdrom

注：建立一个目录用来作挂接点(mount point)

> #mount -o loop -t iso9660 /home/sunky/mydisk.iso /mnt/vcdrom

注：使用/mnt/vcdrom就可以访问盘镜像文件mydisk.iso里的所有文件了。

> [root@CentOS4 /]# mount -t auto /dev/cdrom /mnt/cdrom

**二.挂接移动硬盘**

对linux系统而言，USB接口的移动硬盘是当作SCSI设备对待的。插入移动硬盘之前，应先用fdisk –l 或 more /proc/partitions查看系统的硬盘和硬盘分区情况。

> [root at pldyrouter /]# fdisk -l
>
> Disk /dev/sda: 73 dot 4 GB, 73407820800 bytes
>
> 255 heads, 63 sectors/track, 8924 cylinders
>
> Units = cylinders of 16065 * 512 = 8225280 bytes
>
> Device Boot Start End Blocks Id System
>
> /dev/sda1 1 4 32098+ de Dell Utility
>
> /dev/sda2 * 5 2554 20482875 7 HPFS/NTFS
>
> /dev/sda3 2555 7904 42973875 83 Linux
>
> /dev/sda4 7905 8924 8193150 f Win95 Ext’d (LBA)
>
> /dev/sda5 7905 8924 8193118+ 82 Linux swap

在这里可以清楚地看到系统有一块SCSI硬盘/dev/sda和它的四个磁盘分区/dev/sda1 — /dev/sda4, /dev/sda5是分区/dev/sda4的逻辑分区。接好移动硬盘后，再用fdisk –l 或 more /proc/partitions查看系统的硬盘和硬盘分区情况

> [root at pldyrouter /]# fdisk -l
>
> Disk /dev/sda: 73 dot 4 GB, 73407820800 bytes
>
> 255 heads, 63 sectors/track, 8924 cylinders
>
> Units = cylinders of 16065 * 512 = 8225280 bytes
>
> Device Boot Start End Blocks Id System
>
> /dev/sda1 1 4 32098+ de Dell Utility
>
> /dev/sda2 * 5 2554 20482875 7 HPFS/NTFS
>
> /dev/sda3 2555 7904 42973875 83 Linux
>
> /dev/sda4 7905 8924 8193150 f Win95 Ext’d (LBA)
>
> /dev/sda5 7905 8924 8193118+ 82 Linux swap
>
> Disk /dev/sdc: 40.0 GB, 40007761920 bytes
>
> 255 heads, 63 sectors/track, 4864 cylinders
>
> Units = cylinders of 16065 * 512 = 8225280 bytes
>
> Device Boot Start End Blocks Id System
>
> /dev/sdc1 1 510 4096543+ 7 HPFS/NTFS
>
> /dev/sdc2 511 4864 34973505 f Win95 Ext’d (LBA)
>
> /dev/sdc5 511 4864 34973473+ b Win95 FAT32

大家应该可以发现多了一个SCSI硬盘/dev/sdc和它的两个磁盘分区/dev/sdc1?、/dev/sdc2,其中/dev/sdc5是/dev/sdc2分区的逻辑分区。我们可以使用下面的命令挂接/dev/sdc1和/dev/sdc5。

> #mkdir -p /mnt/usbhd1
> #mkdir -p /mnt/usbhd2

注：建立目录用来作挂接点(mount point)

#mount -t ntfs /dev/sdc1 /mnt/usbhd1

#mount -t vfat /dev/sdc5 /mnt/usbhd2

注：对ntfs格式的磁盘分区应使用-t ntfs 参数，对fat32格式的磁盘分区应使用-t vfat参数。若汉字文件名显示为乱码或不显示，可以使用下面的命令格式。

#mount -t ntfs -o iocharset=cp936 /dev/sdc1 /mnt/usbhd1

#mount -t vfat -o iocharset=cp936 /dev/sdc5 /mnt/usbhd2

linux系统下使用fdisk分区命令和mkfs文件系统创建命令可以将移动硬盘的分区制作成linux系统所特有的ext2、ext3格式。这样，在linux下使用就更方便了。使用下面的命令直接挂接即可。

#mount /dev/sdc1 /mnt/usbhd1



**三.挂接U盘**

和USB接口的移动硬盘一样对linux系统而言U盘也是当作SCSI设备对待的。使用方法和移动硬盘完全一样。插入U盘之前，应先用fdisk –l 或 more /proc/partitions查看系统的硬盘和硬盘分区情况。

> [root at pldyrouter root]# fdisk -l
>
> Disk /dev/sda: 73 dot 4 GB, 73407820800 bytes
>
> 255 heads, 63 sectors/track, 8924 cylinders
>
> Units = cylinders of 16065 * 512 = 8225280 bytes
>
> Device Boot Start End Blocks Id System
>
> /dev/sda1 1 4 32098+ de Dell Utility
>
> /dev/sda2 * 5 2554 20482875 7 HPFS/NTFS
>
> /dev/sda3 2555 7904 42973875 83 Linux
>
> /dev/sda4 7905 8924 8193150 f Win95 Ext’d (LBA)
>
> /dev/sda5 7905 8924 8193118+ 82 Linux swap

插入U盘后，再用fdisk –l 或 more /proc/partitions查看系统的硬盘和硬盘分区情况。

> [root at pldyrouter root]# fdisk -l
>
> Disk /dev/sda: 73 dot 4 GB, 73407820800 bytes
>
> 255 heads, 63 sectors/track, 8924 cylinders
>
> Units = cylinders of 16065 * 512 = 8225280 bytes
>
> Device Boot Start End Blocks Id System
>
> /dev/sda1 1 4 32098+ de Dell Utility
>
> /dev/sda2 * 5 2554 20482875 7 HPFS/NTFS
>
> /dev/sda3 2555 7904 42973875 83 Linux
>
> /dev/sda4 7905 8924 8193150 f Win95 Ext’d (LBA)
>
> /dev/sda5 7905 8924 8193118+ 82 Linux swap
>
> Disk /dev/sdd: 131 MB, 131072000 bytes
>
> 9 heads, 32 sectors/track, 888 cylinders
>
> Units = cylinders of 288 * 512 = 147456 bytes
>
> Device Boot Start End Blocks Id System
>
> /dev/sdd1 * 1 889 127983+ b Win95 FAT32
>
> Partition 1 has different physical/logical endings:
>
> phys=(1000, 8, 32) logical=(888, 7, 31)

系统多了一个SCSI硬盘/dev/sdd和一个磁盘分区/dev/sdd1,/dev/sdd1就是我们要挂接的U盘。

> #mkdir -p /mnt/usb

注：建立一个目录用来作挂接点(mount point)

> #mount -t vfat /dev/sdd1 /mnt/usb

注：现在可以通过/mnt/usb来访问U盘了, 若汉字文件名显示为乱码或不显示，可以使用下面的命令。

> #mount -t vfat -o iocharset=cp936 /dev/sdd1 /mnt/usb

**四.挂接Windows文件共享**

Windows网络共享的核心是SMB/CIFS，在linux下要挂接(mount)windows的磁盘共享，就必须安装和使用samba 软件包。现在流行的linux发行版绝大多数已经包含了samba软件包，如果安装linux系统时未安装samba请首先安装samba。当然也可以到 [www.samba.org][1]网站下载……新的版本是3.0.10版。

当windows系统共享设置好以后，就可以在linux客户端挂接(mount)了，具体操作如下：

> \# mkdir –p /mnt/samba

注：建立一个目录用来作挂接点(mount point)

> \# mount -t smbfs -o username=administrator,password=pldy123 //10.140.133.23/c$ /mnt/samba

注：administrator 和 pldy123 是ip地址为10.140.133.23 windows计算机的一个用户名和密码，c$是这台计算机的一个磁盘共享

如此就可以在linux系统上通过/mnt/samba来访问windows系统磁盘上的文件了。以上操作在redhat as server 3、redflag server 4.1、suse server 9以及windows NT 4.0、windows 2000、windows xp、windows 2003环境下测试通过。

**五.挂接UNIX系统NFS文件共享**

类似于windows的网络共享，UNIX(Linux)系统也有自己的网络共享，那就是NFS(网络文件系统)，下面我们就以SUN Solaris2.8和REDHAT as server 3 为例简单介绍一下在linux下如何mount nfs网络共享。

在linux客户端挂接(mount)NFS磁盘共享之前，必须先配置好NFS服务端。

1、Solaris系统NFS服务端配置方法如下：

(1)修改 /etc/dfs/dfstab, 增加共享目录

> share -F nfs -o rw /export/home/sunky

(2)启动nfs服务

> \# /etc/init.d/nfs.server start

(3)NFS服务启动以后，也可以使用下面的命令增加新的共享

> \# share /export/home/sunky1
> \# share /export/home/sunky2

注：/export/home/sunky和/export/home/sunky1是准备共享的目录

2、linux系统NFS服务端配置方法如下：

(1)修改 /etc/exports,增加共享目录

> /export/home/sunky 10.140.133.23(rw)
>
> /export/home/sunky1 *(rw)
>
> /export/home/sunky2 linux-client(rw)

注：/export/home/目录下的sunky、sunky1、sunky2是准备共享的目录，10.140.133.23、*、 linux-client是被允许挂接此共享linux客户机的IP地址或主机名。如果要使用主机名linux-client必须在服务端主机 /etc/hosts文件里增加linux-client主机ip定义。格式如下：

> 10.140.133.23 linux-client

(2)启动与停止NFS服务

> /etc/rc.d/init.d/portmap start (在REDHAT中PORTMAP是默认启动的)
>
> /etc/rc.d/init.d/nfs start 启动NFS服务
>
> /etc/rc.d/init.d/nfs stop 停止NFS服务

注：若修改/etc/export文件增加新的共享，应先停止NFS服务，再启动NFS服务方能使新增加的共享起作用。使用命令exportfs -rv也可以达到同样的效果。

3、linux客户端挂接(mount)其他linux系统或UNIX系统的NFS共享

> \# mkdir –p /mnt/nfs

注：建立一个目录用来作挂接点(mount point)

> #mount -t nfs -o rw 10.140.133.9:/export/home/sunky /mnt/nfs

注：这里我们假设10.140.133.9是NFS服务端的主机IP地址，当然这里也可以使用主机名，但必须在本机/etc/hosts文件里增加服务端ip定义。/export/home/sunky为服务端共享的目录。

如此就可以在linux客户端通过/mnt/nfs来访问其它linux系统或UNIX系统以NFS方式共享出来的文件了。以上操作在 redhat as server 3、redflag server4.1、suse server 9以及Solaris 7、Solaris 8、Solaris 9 for x86&sparc环境下测试通过。

权限问题：

假設 server 端的使用者 jack, user id 為 1818, gid 為 1818, client 端也有一個使用者 jack，但是 uid 及 gid 是 1818。client 端的 jack    希望能完全讀寫 server 端的 /home/jack 這個目錄。server 端的 /etc/exports 是

這樣寫的：

> /home/jack *(rw,all_squash,anonuid=1818,anongid=1818)

這個的設定檔的意思是，所有 client 端的使用者存取 server 端 /home/jack 這

目錄時，都會 map 成 server 端的 jack (uid,gid=1818)。我 mount 的結果是

1. client 端的 root 可以完全存取該目錄, 包括讀、寫、殺……等

2. client 端的 jack (uid,gid=1818) 我可以做：

> rm -rf server_jack/*
> cp something server_jack/
> mkdir server_jack/a



 [1]: http://www.samba.org/