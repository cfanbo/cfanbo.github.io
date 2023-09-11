---
title: 为VMware Linux增加虚拟硬盘
author: admin
type: post
date: 2011-08-31T13:06:44+00:00
url: /archives/11082
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux
 - vmware

---
VMware安装Linux的时候默认分配的空间是4GB，可能会不够，这个时候可以通过增加一块虚拟硬盘，将/usr或其他内容拷贝过去解决这个问题：

**总个操作过程可分为：**

 1. 分区
 2. 格式化
 3. 挂载

三个过程.

**创建虚拟硬盘**
1、关闭VM中正在运行的虚拟系统；

2、在虚拟系统名称上点右键－》Virtual Machine Settings；
3、在Hardware页点“Add”－》Add a hard disk－》Create a new virtual disk－》SCSI(recommended)－》分配空间大小－》OK；
4、可以看见Hardware中出现了一块新的硬盘Hard Disk 2。

**对虚拟硬盘进行分区和格式化**
[root@cncmail data1]# fdisk -l ## 这里是查看目前系统上有几块硬盘

Disk /dev/sda: 36.4 GB, 36401479680 bytes
255 heads, 63 sectors/track, 4425 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes

Device Boot     Start        End     Blocks    Id   System
/dev/sda1    *          1        255    2048256    83   Linux
/dev/sda2            256       1530   10241437+   83   Linux
/dev/sda3           4296       4425    1044225    82   Linux swap
/dev/sda4           1531       4295   22209862+    f   Win95 Ext’d (LBA)
/dev/sda5           1531       2805   10241406    83   Linux
/dev/sda6           2806       4295   11968393+   83   Linux

Partition table entries are not in disk order

Disk /dev/sdb: 36.7 GB, 36703918080 bytes ## 这里发现/dev/sdb，容量36.7G，切未被分区
255 heads, 63 sectors/track, 4462 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes

Disk /dev/sdc doesn’t contain a valid partition table
[root@linux root]# fdisk /dev/sdb ## 接下去就对/dev/sdb分区进行分区

The number of cylinders for this disk is set to 4462.
There is nothing wrong with that, but this is larger than 1024,
and could in certain setups cause problems with:
1) software that runs at boot time (e.g., old versions of LILO)
2) booting and partitioning software from other OSs
(e.g., DOS FDISK, OS/2 FDISK)

Command (m for help): m
Command action
a    toggle a bootable flag
b    edit bsd disklabel
c    toggle the dos compatibility flag
d    delete a partition
l    list known partition types
m    print this menu
n    add a new partition
o    create a new empty DOS partition table
p    print the partition table
q    quit without saving changes
s    create a new empty Sun disklabel
t    change a partition’s system id
u    change display/entry units
v    verify the partition table
w    write table to disk and exit
x    extra functionality (experts only)

Command (m for help): p  ## 打印出目前该硬盘下的分区列表

Disk /dev/sdb: 36.7 GB, 36703918080 bytes
255 heads, 63 sectors/track, 4462 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes

Device Boot     Start        End     Blocks    Id   System

Command (m for help): n    ## 增加一个分区
Command action
e    extended
p    primary partition (1-4)
\## 因为通常选择主分区，所以这里打一个p
pPartition number (1-4): 1    ## 这里因为是第一个分却，所以只选择1，如果是第二个分区，则选择2,依次类推
First cylinder (1-4462, default 1): ## 新分区起始的磁盘块数
Using default value 1
Last cylinder or +size or +sizeM or +sizeK (1-4462, default 4462): 如果要分区10G，这里可以直接输入：+10240M，因为这里要全部使用硬盘空间，则用默认
Using default value 4462

Command (m for help): p

Disk /dev/sdb: 36.7 GB, 36703918080 bytes
255 heads, 63 sectors/track, 4462 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes

Device Boot     Start        End     Blocks    Id   System
/dev/sdb1              1       4462   35840983+   83   Linux

\## 这里第一个分区已经分好了，接下去得把这个分区写入硬盘，用w
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.

下面的工作就是对该硬盘进行格式，我这里是格式化成ext3
[root@linux root]# mkfs.ext3 /dev/sdb1 (这里原来的命令是：mke2fs -j /dev/sdb1，试了一下不成功，改了)

> mke2fs 1.32 (09-Nov-2002)
> Filesystem label=
> OS type: Linux
> Block size=4096 (log=2)
> Fragment size=4096 (log=2)
> 4480448 inodes, 8960245 blocks
> 448012 blocks (5.00%) reserved for the super user
> First data block=0
> 274 block groups
> 32768 blocks per group, 32768 fragments per group
> 16352 inodes per group
> Superblock backups stored on blocks:
> 32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
> 4096000, 7962624
>
> Writing inode tables: done
> Creating journal (8192 blocks): done
> Writing superblocks and filesystem accounting information: done
>
> This filesystem will be automatically checked every 23 mounts or
> 180 days, whichever comes first.   Use tune2fs -c or -i to override.

检查一下，是否已经格式好
[root@linux root]# fdisk -l

> Disk /dev/sda: 36.4 GB, 36401479680 bytes
>
> 255 heads, 63 sectors/track, 4425 cylinders
> Units = cylinders of 16065 * 512 = 8225280 bytes
>
> Device Boot     Start        End     Blocks    Id   System
> /dev/sda1    *          1        255    2048256    83   Linux
> /dev/sda2            256       1530   10241437+   83   Linux
> /dev/sda3           4296       4425    1044225    82   Linux swap
> /dev/sda4           1531       4295   22209862+    f   Win95 Ext’d (LBA)
> /dev/sda5           1531       2805   10241406    83   Linux
> /dev/sda6           2806       4295   11968393+   83   Linux
>
> Partition table entries are not in disk order
>
> Disk /dev/sdb: 36.7 GB, 36703918080 bytes
> 255 heads, 63 sectors/track, 4462 cylinders
> Units = cylinders of 16065 * 512 = 8225280 bytes
>
> Device Boot     Start        End     Blocks    Id   System
> /dev/sdb1              1       4462   35840983+   83   Linux

**挂载虚拟硬盘**
分区分好，也格式化好了，下面就是挂载
我把/dev/sdb1挂载到/data1下
[root@linux root]# mkdir /data1   ## 首先建立挂载的目录data1
[root@linux root]# mount /dev/sdb1 /data1 ##将sdb1挂载到data1

重启系统之后，查看是否挂载成功：
[root@linux data1]# df -h
文件系统               容量   已用 可用 已用% 挂载点
/dev/sda1              2.0G   454M   1.4G   25% /
/dev/sda6               12G    53M    11G    1% /bak
/dev/sdb1               34G    33M    32G    1% /data1
none                   250M      0   250M    0% /dev/shm
/dev/sda2              9.7G   1.5G   7.7G   17% /usr
/dev/sda5              9.7G   8.6G   559M   95% /var

这里看到/dev/sda6               12G    53M    11G    1% /bak
说明已经挂载成功了。到根目录“/”下可以查看到这个挂载好的data1。

**转移数据**
其实一直做到这里都还只是准备工作，如果根分区下的数据不转移到这个虚拟硬盘中的话，还是会提示空间不足。下面是将/usr全部转移到虚拟硬盘中的过程（参考Linux人生的《Linux系统精华之一——挂载》），同样也可以转移其他目录：

1、将/usr中的全部数据拷贝到data1（可以用mv一个一个拷贝，也可以用tar压缩之后一次拷贝，具体参见这两个命令的man）
2、清空usr目录：
\# rm -r /usr
\# mkdir /usr
3、卸载刚才挂上的虚拟硬盘，重新将它挂载到usr目录：
\# umount /dev/sdb1 /data1
\# mount /dev/sdb1 /usr
4、# vi /etc/fstab ## 用vi修改/etc/fstab，使系统启动就可以自动挂载
（点击“i”进入插入模式对文本内容进行修改，改好后点“Esc”，输入冒号“:”进入命令行模式，输入wq保存退出，具体操作可以参考vi常用指令）

在内容中加上一行：
/dev/sdb1                /usr                     ext3     defaults         1 2

4、Ok，重新启动之后，可以查看现在的硬盘使用情况了：
\# df -h
文件系统               容量   已用 可用 已用% 挂载点
/dev/sda2              3.6G   1.3G   2.4G   35% /
udev                   125M   124K   125M    1% /dev
/dev/sdb1              4.0G   2.3G   1.6G   60% /usr
根分区的“已用%”从99%降到了35%，哈哈，大功告成，可以继续做其他的事情了。不过这次添加的虚拟硬盘还是比较小，完全可以在添加的时候设得大一点的。美中不足。

**附录：**

**Linux下/etc/fstab文件详解**

有很多人经常修改/etc/fstab文件，但是其中却有很多人对这个文件所表达的意义不太清楚，因为只要按照一定的模式，就可以轻而易举地添加一行挂载信息，而不需要完全理解其中的原理。下面就让我们来看看到底还有多少是我们不了解的。
/etc/fstab是用来存放文件系统的静态信息的文件。位于/etc/目录下，可以用命令less /etc/fstab 来查看，如果要修改的话，则用命令 vi /etc/fstab 来修改。
当系统启动的时候，系统会自动地从这个文件读取信息，并且会自动将此文件中指定的文件系统挂载到指定的目录。下面我来介绍如何在此文件下填写信息。
在这个文件下，我们要关注的是它的六个域，分别为：、、 、、、。下面将详细介绍这六个域的详细意义。
1、。这里用来指定你要挂载的文件系统的设备名称或块信息，也可以是远程的文件系统。做过嵌入式linux开发的朋友都可能知道 mount 192.168.1.56:/home/nfs /mnt/nfs/ -o nolock (可以是其他IP)命令所代表的意义。它的任务是把IP为192.168.1.56的远程主机上的/home/nfs/目录挂载到本机的/mnt/nfs /目录之下。如果要把它写进/etc/fstab文件中，file system这部分应填写为：/192.168.1.56:/home/nfs/。
如果想把本机上的某个设备（device）挂载上来，写法如：/dev/sda1、/dev/hda2或/dev/cdrom，其中，/dev/sda1 表示第一个串口硬盘的第一个分区，也可以是第一个SCSI硬盘的第一个分区，/dev/hda1表示第一个IDE硬盘的第一个分区，/dev/cdrom 表示光驱。
此外，还可以label(卷标)或UUID（Universally Unique Identifier全局唯一标识符）来表示。用label表示之前，先要e2label创建卷标，如：e2label /dir\_1 /dir\_2，其意思是说用/dir\_2来表示/dir\_1的名称。然后，再在/etc/fstab下添加：LABEL=/dir\_2 /dir\_2 。重启后，系统就会将/dir\_1挂载到/dir\_2目录上。对于UUID，可以用vol\_id -u /dev/sdax来获取。比如我想挂载第一块硬盘的第一个分区，先用命令vol\_id -u /dev/sda11 来取得UUID，比如是：5dc08a62-3472-471b-9ef5-0a91e5e2c126，然后在这个域上填写： UUID=5dc08a62-3472-471b-9ef5-0a91e5e2c126，即可表示/dev/sda11。Red Hat linux 一般会使用label，而Ubuntu linux 一般会用UUID。
2、。挂载点，也就是自己找一个或创建一个dir（目录），然后把文件系统挂到这个目录上，然后就可以从这个目录中访问要挂载文件系统。对于swap分区，这个域应该填写：none，表示没有挂载点。
3、。这里用来指定文件系统的类型。下面的文件系统都是目前Linux所能支持的：adfs、befs、cifs、ext3、 ext2、ext、iso9660、kafs、minix、msdos、vfat、umsdos、proc、reiserfs、swap、 squashfs、nfs、hpfs、ncpfs、ntfs、affs、ufs。
4、。这里用来填写设置选项，各个选项用逗号隔开。由于选项非常多，而这里篇幅有限，所以不再作详细介绍，如需了解，请用 命令 man mount 来查看。但在这里有个非常重要的关键字需要了解一下：defaults，它代表包含了选项rw,suid,dev,exec,auto,nouser和 async。
5、。此处为1的话，表示要将整个里的内容备份；为0的话，表示不备份。现在很少用到dump这个工具，在这里一般选0。
6、。这里用来指定如何使用fsck来检查硬盘。如果这里填0，则不检查；挂载点为 / 的（即根分区），必须在这里填写1，其他的都不能填写1。如果有分区填写大于1的话，则在检查完根分区后，接着按填写的数字从小到大依次检查下去。同数字 的同时检查。比如第一和第二个分区填写2，第三和第四个分区填写3，则系统在检查完根分区后，接着同时检查第一和第二个分区，然后再同时检查第三和第四个 分区。

**参考文献：**
1、On-line reference manuals of Linux (用命令 man 5 fstab 查看)。
2、_**Linux Bible 2008 Edition**._   By Christopher Negus. Published by Wiley Publishing, Inc.2008
3、**_Linux Administration Handbook_** (Second Edition)    By [US] Evi Nemeth   Garth Snyder   Trent R. Hein .    Published by Pearson Education,Inc.2007