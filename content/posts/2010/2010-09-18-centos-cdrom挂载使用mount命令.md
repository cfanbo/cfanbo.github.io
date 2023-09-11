---
title: CentOS CDROM挂载使用mount命令
author: admin
type: post
date: 2010-09-18T13:20:11+00:00
url: /archives/5735
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - mount

---
CentOS CDROM挂载还是比较常用的，于是我研究了一下CentOS CDROM挂载，在这里拿出来和大家分享一下，希望CentOS CDROM挂载对大家有用。使用mount命令CentOS CDROM挂载学习目的是能访问CentOS CDROM挂载中的数据。

Linux显示所有的目录都在一个目录树下，而于他们位于哪一个驱动器/硬件无关。在Linux下的磁盘内容作为子目录形式出现的。可移动介质的内容不会自动出现在这些自目录的，我们必须通过挂载驱动器来实现。

**用mount命令来挂载CentOS CDROM挂载.**

命令：mount -t auto /dev/cdrom /mnt/cdrom

这命令就是把CentOS CDROM挂载在/mnt/cdrom目录中，这里我就可以访问里面的内容了。

学习操作过程：

[OK_008@CentOS4 ~]$ mount -t auto /dev/cdrom /mnt/cdrommount: only root can do that

–一般用户无法挂载cdrom,只有root用户才可以操作。

[OK_008@CentOS4 ~]$ –切换用户操作：

[root@CentOS4 /]# mount -t auto /dev/cdrom /mnt/cdrommount: mount point /mnt/cdrom does not exist –/mnt/cdrom目录不存在，需要先创建。

[root@CentOS4 /]# cd /mnt-bash: cd: /mnt: No such file or directory[root@CentOS4 /]

\# [root@CentOS4 /]# mkdir -p /mnt/cdrom  –创建/mnt/cdrom目录[root@CentOS4 /]# lsbin   dev  home    lib         media  mnt  proc  sbin     srv  tmp  varboot  etc  initrd  lost+found  misc   opt  root  selinux  sys  usr
[root@CentOS4 /]# mount -t auto /dev/cdrom /mnt/cdrom  –挂载CentOS CDROM挂载mount: block device /dev/cdrom is write-protected, mounting read-only –挂载成功。

[root@CentOS4 /]# ls -l /mnt/cdrom –查看CentOS CDROM挂载里面内容total 859
dr-xr-xr-x  4 root root   2048 Sep  4  2005 CentOS
-r–r–r–  2 root root   8859 Mar 19  2005 centosdocs-man.css
-r–r–r–  9 root root  18009 Mar  1  2005 GPL
dr-xr-xr-x  2 root root 241664 May  7 02:32 headers
dr-xr-xr-x  4 root root   2048 May  7 02:23 images
dr-xr-xr-x  2 root root   4096 May  7 02:23 isolinux
dr-xr-xr-x  2 root root  18432 May  2 18:50 NOTES
-r–r–r–  2 root root   5443 May  7 01:49 RELEASE-NOTES-en.html
dr-xr-xr-x  2 root root   2048 May  7 02:34 repodata
-r–r–r–  9 root root   1795 Mar  1  2005 RPM-GPG-KEY
-r–r–r–  2 root root   1795 Mar  1  2005 RPM-GPG-KEY-centos4
-r–r–r–  1 root root 571730 May  7 01:39 yumgroups.xml
[root@CentOS4 /]#

[root@CentOS4 /]# umount /mnt/cdrom

–卸载CentOS CDROM挂载,很容易，直接使用umount /mnt/cdrom 即可。

**另mount命令其他参数说明可以参考如下：**

名称 : mount

使用权限 : 系统管理者或/etc/fstab中允许的使用者

使用方式 : mount \[-hV] mount -a [-fFnrsvw\] \[-t vfstype\] mount \[-fnrsvw\] \[-o options [,…\]] device | dir mount \[-fnrsvw\] \[-t vfstype\] [-o options] device dir

说明 : 将某个档案的内容解读成档案系统，然后将其挂在目录的某个位置之上。当这个命令执行成功后，直到我们使用 umnount 将这个档案系统移除为止，这个命令之下的所有档案将暂时无法被调用。这个命令可以被用来挂上任何的档案系统，你甚至可以用 -o loop 选项将某个一般的档案当成硬盘机分割挂上系统。这个功能对于 ramdisk,romdisk 或是 ISO 9660 的影像档之解读非常实用。

**参数**
-V 显示程序版本
-h 显示辅助讯息
-v 显示较讯息，通常和 -f 用来除错。
-a 将 /etc/fstab 中定义的所有档案系统挂上。
-F 这个命令通常和 -a 一起使用，它会为每一个 mount 的动作产生一个行程负责执行。在系统需要挂上大量 NFS 档案系统时可以加快挂上的动作。
-f 通常用在除错的用途。它会使 mount 并不执行实际挂上的动作，而是模拟整个挂上的过程。通常会和 -v 一起使用。
-n 一般而言，mount 在挂上后会在 /etc/mtab 中写入一笔资料。但在系统中没有可写入档案系统存在的情况下可以用这个选项取消这个动作。

-s-r 等于 -o ro -w 等于 -o rw -L 将含有特定标签的硬盘分割挂上。
-U 将档案分割序号为 的档案系统挂下。
-L 和 -U 必须在/proc/partition 这种档案存在时才有意义。
-t 指定档案系统的型态，通常不必指定。mount 会自动选择正确的型态。
-o async 打开非同步模式，所有的档案读写动作都会用非同步模式执行。 -o sync 在同步模式下执行。
-o atime -o noatime 当 atime 打开时，系统会在每次读取档案时更新档案的『上一次调用时间』。

当我们使用 flash 档案系统时可能会选项把这个选项关闭以减少写入的次数。
-o auto -o noauto 打开/关闭自动挂上模式。 -o defaults 使用预设的选项 rw, suid, dev, exec, auto, nouser, and async.
-o dev -o nodev-o exec -o noexec 允许执行档被执行。
-o suid -o nosuid 允许执行档在 root 权限下执行。
-o user -o nouser 使用者可以执行 mount/umount 的动作。
-o remount 将一个已经挂下的档案系统重新用不同的方式挂上。例如原先是唯读的系统，现在用可读写的模式重新挂上。
-o ro 用唯读模式挂上。
-o rw 用可读写模式挂上。
-o loop= 使用 loop 模式用来将一个档案当成硬盘分割挂上系统。

**范例:**
将 /dev/hda1 挂在 /mnt 之下。
#mount /dev/hda1 /mnt

将 /dev/hda1 用唯读模式挂在 /mnt 之下。
#mount -o ro /dev/hda1 /mnt

将 /tmp/image.iso 这个光碟的 image 档使用 loop 模式挂在 /mnt/cdrom之下。用这种方法可以将一般网络上可以找到的 Linux 光 碟 ISO 档在不烧录成光碟的情况下检视其内容。
#mount -o loop /tmp/image.iso /mnt/cdrom 相关命令umount 。

有关mount的用法请参考: