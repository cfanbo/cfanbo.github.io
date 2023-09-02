---
title: 在 vmware 里扩容Linux硬盘空间
author: admin
type: post
date: 2022-10-31T07:55:46+00:00
url: /archives/31973
categories:
 - 其它
tags:
 - vmware

---
在mac上创建了一个ubuntu的虚拟机做为k8s远程开发调试机器，但在编译程序的时候发现磁盘空间不足，于是需要对磁盘进行扩容。

下面为整个硬盘扩容过程。

# 硬盘扩容 

首先将虚拟机关机，在 Vmware Fusion 软件上面菜单，点击 `Virtual Machine` -> `Setting` 菜单，然后在突出框框里的找到硬盘 `Hard Disk` ，点击调整硬盘空间大小，然后点击`Apply` 应用按钮。![image-20230728130214297](https://blogstatic.haohtml.com/uploads/2022/10/68da0b3518a7ced98a9fd17cb55b89dd-1.webp)

# 磁盘格式化 

查看当前磁盘分区信息

```
sxf@vm:~/workspace$ sudo fdisk -l
[sudo] password for sxf:
Disk /dev/loop0: 63.29 MiB, 66359296 bytes, 129608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/loop1: 63.46 MiB, 66531328 bytes, 129944 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/loop2: 91.85 MiB, 96292864 bytes, 188072 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/loop3: 49.86 MiB, 52260864 bytes, 102072 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/loop4: 53.26 MiB, 55844864 bytes, 109072 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

## 注意这里由于/dev/sda 磁盘进行了扩容，因此会提示不匹配
GPT PMBR size mismatch (41943039 != 104857599) will be corrected by write.
Disk /dev/sda: 50 GiB, 53687091200 bytes, 104857600 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 433D7B47-01B6-4710-AFAB-7709921A5BBF

Device       Start      End  Sectors  Size Type
/dev/sda1     2048     4095     2048    1M BIOS boot
/dev/sda2     4096  3719167  3715072  1.8G Linux filesystem
/dev/sda3  3719168 41940991 38221824 18.2G Linux filesystem

Disk /dev/mapper/ubuntu--vg-ubuntu--lv: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

查看磁盘 `/dev/sda` 信息

```
sxf@vm:~/workspace$ sudo fdisk /dev/sda

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

GPT PMBR size mismatch (41943039 != 104857599) will be corrected by write.

# 查看命令帮助信息
Command (m for help): m

Help:

  GPT
   M   enter protective/hybrid MBR

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition

  Misc
   m   print this menu
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty DOS partition table
   s   create a new empty Sun partition table

Command (m for help):
```

我们将用到 分区表（ `p`） 、添加分区(`n`) 和 保存分区表（ `w`） 三个命令。

## 打印分区 

```
# 查看当前分区表
Command (m for help): p

Disk /dev/sda: 50 GiB, 53687091200 bytes, 104857600 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 433D7B47-01B6-4710-AFAB-7709921A5BBF

Device       Start      End  Sectors  Size Type
/dev/sda1     2048     4095     2048    1M BIOS boot
/dev/sda2     4096  3719167  3715072  1.8G Linux filesystem
/dev/sda3  3719168 41940991 38221824 18.2G Linux filesystem
```

## 创建新分区 

```
Command (m for help): n
Partition number (4-128, default 4):
First sector (41940992-104857566, default 41940992):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (41940992-104857566, default 104857566):

Created a new partition 4 of type 'Linux filesystem' and of size 30 GiB.
```

中间的提示，直接按回车键即可。

> 这里新扩容了 30G 硬盘空间，分区编号为 4，即分区名为 /dev/sda4

## 保存分区配置 

输出命令 `w` 保存分区信息

```
Command (m for help): w

The partition table has been altered.
Syncing disks.
```

进行确认分区是否成功

```
sxf@vm:~/workspace$ sudo fdisk -l
Disk /dev/loop0: 63.29 MiB, 66359296 bytes, 129608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/loop1: 63.46 MiB, 66531328 bytes, 129944 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/loop2: 91.85 MiB, 96292864 bytes, 188072 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/loop3: 49.86 MiB, 52260864 bytes, 102072 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/loop4: 53.26 MiB, 55844864 bytes, 109072 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sda: 50 GiB, 53687091200 bytes, 104857600 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 433D7B47-01B6-4710-AFAB-7709921A5BBF

Device        Start       End  Sectors  Size Type
/dev/sda1      2048      4095     2048    1M BIOS boot
/dev/sda2      4096   3719167  3715072  1.8G Linux filesystem
/dev/sda3   3719168  41940991 38221824 18.2G Linux filesystem
/dev/sda4  41940992 104857566 62916575   30G Linux filesystem

Disk /dev/mapper/ubuntu--vg-ubuntu--lv: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

此时多出一个 `/dev/sda4` 分区，其中新分区的起始位置(`41940992`)正是 `/dev/sda3`(`41940991`) 的结束位置。

然后重启机器。

# 挂载分区 

上面我们刚刚创建了一个新的分区，在使用前，需要经过两个步骤，首先是对分区进行初始化，然后将分区挂载到一个目录里，后续就可以直接这个目录进行创建文件或文件夹之类的操作了。

## 格式化分区 

```
sxf@vm:~$ sudo mkfs.ext4 /dev/sda4
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 7864320 4k blocks and 1969920 inodes
Filesystem UUID: c6885c8e-b93a-4e69-b898-122e9b3b0139
Superblock backups stored on blocks:
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
    4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

## 挂载分区 

```
sxf@vm:~$ sudo mkdir /data
sxf@vm:~$ sudo mount /dev/sda4 /data
sxf@vm:~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
udev                               1.9G     0  1.9G   0% /dev
tmpfs                              389M  1.6M  388M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  9.8G  7.7G  1.6G  83% /
tmpfs                              1.9G     0  1.9G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
tmpfs                              1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/loop0                          64M   64M     0 100% /snap/core20/1828
/dev/loop1                          50M   50M     0 100% /snap/snapd/18357
/dev/loop3                          64M   64M     0 100% /snap/core20/1974
/dev/loop2                          92M   92M     0 100% /snap/lxd/24061
/dev/loop4                          54M   54M     0 100% /snap/snapd/19457
/dev/sda2                          1.8G  108M  1.5G   7% /boot
tmpfs                              389M     0  389M   0% /run/user/1000
/dev/sda4                           30G   24K   28G   1% /data
```

可以看到最后一行， 表示已经成功挂载 `/dev/sda4` 分区到本机的 `/data` 目录。

## 使用测试 

接着我们创建一些文件，测试一下

```
sxf@vm:~$ ls -al /data
total 24
drwxr-xr-x  3 root root  4096 Jul 28 04:35 .
drwxr-xr-x 20 root root  4096 Jul 28 04:36 ..
drwx------  2 root root 16384 Jul 28 04:35 lost+found

sxf@vm:~$ touch /data/a.txt
touch: cannot touch '/data/a.txt': Permission denied
sxf@vm:~$ sudo touch /data/a.txt
sxf@vm:~$ ls -al /data/a.txt
-rw-r--r-- 1 root root 0 Jul 28 04:37 /data/a.txt
```

可以看到可以正常操作文件，至此扩展完毕。