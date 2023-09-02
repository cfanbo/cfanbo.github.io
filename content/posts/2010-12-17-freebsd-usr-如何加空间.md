---
title: freebsd /usr 如何加空间
author: admin
type: post
date: 2010-12-17T04:24:08+00:00
url: /archives/6973
IM_contentdowned:
 - 1
categories:
 - 服务器

---
ln -s /usr/tmpbak /tmp这样你的/tmp目录就可以使用/usr分区的空间。

1.找到不用的分區或者硬盘
2.newfs /dev/“你的分区或者硬盘”
3.mount /dev/“你的分区或者硬盘” /mnt
4.cd “你要扩大空间的目录”
5.tar cf – * |(cd /mnt ; tar xf -)
6.修改/etc/fstable ，挂載到你要擴展的目錄。
7.reboot

**在添加物理硬盘后操作:**

/stand/sysinstall

选择configure–>进入下一级菜单

选择FDisk–>进入下一级菜单


选择要分区的硬盘;进而磁盘分片界面;
进行分片(create slice)操作;并保存W(write);
系统提示选择磁盘加载模式,选择”standard”

选择Disklabel–>进而磁盘分区界面;
C(Create)创建分区;
M (M = Mount pt.)定义分区的加载点;     #这步非常关键!
W (write);存盘                         #根据提示选择Yes,系统会调用Newfs进行分区;成功后用df -h查看可以发现新分区已经加载;
#步骤安装系统时的分区过程;要记录下磁盘设备名和加载点;

#注意:如果你的系统以前有一个分区为/opt/data,现在要加载一个分区为/opt/data1,在此步骤操作中,系统界面mount项只会显示/dev/ad1s1e    /opt/data;故此,定义分区的加载点的步骤是否关键,容易使管理员在视觉上混淆概念;

更改分区表文件
vi /etc/fstab
\# See the fstab(5) manual page for important information on automatic mounts
\# of network filesystems before modifying this file.
#
\# Device                  Mountpoint        FStype    Options           Dump      Pass#
/dev/ad0s1b               none              swap      sw                0         0
/dev/ad0s1a               /                 ufs       rw                1         1
/dev/ad0s1h               /opt              ufs       rw                2         2
/dev/ad0s1d               /opt/data                 ufs       rw                2         2
/dev/ad0s1g               /tmp              ufs       rw                2         2
/dev/ad0s1e               /usr              ufs       rw                2         2
/dev/ad0s1f               /var              ufs       rw                2         2
/dev/acd0c                /cdrom            cd9660    ro,noauto         0         0
proc                      /proc             procfs    rw                0         0
#添加如下这条,输入磁盘设备名,加载点,文件类型,读写权限,启动级别等;
/dev/ad1s1e               /opt/data1        ufs       rw                3         3

摘自: