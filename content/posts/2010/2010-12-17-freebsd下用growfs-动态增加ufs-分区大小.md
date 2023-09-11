---
title: freebsd下用growfs 动态增加UFS 分区大小
author: admin
type: post
date: 2010-12-17T04:07:03+00:00
url: /archives/6959
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - growfs
 - 分区

---
/data 不够用了，咋办？

[root@mercury8] ~# /usr/local/etc/rc.d/nginx stop

代码:


```
Stopping nginx.
```

[root@mercury8] ~# umount /data

[root@mercury8] ~# fdisk -BI da1

代码:

```
******* Working on device /dev/da1 *******  fdisk: Class not found
```

用sysinstall 的 fdisk 察看能扩展到哪个扇区：超出没关系，会提示你正确的最大值。

引用:


Disk name: da1 FDISK Partition Editor

DISK Geometry: 5874 cyls/255 heads/63 sectors = 94365810 sectors (46077MB)


Offset Size(ST) End Name PType Desc Subtype Flags


0 63 62 – 12 unused 0


63 94365747 94365809 da1s1 8 freebsd 165


94365810 6030 94371839 – 12 unused 0


The following commands are supported (in upper or lower case):


A = Use Entire Disk G = set Drive Geometry C = Create Slice


D = Delete Slice Z = Toggle Size Units S = Set Bootable | = Expert m.


T = Change Type U = Undo All Changes W = Write Changes


Use F1 or ? to get more help, arrow keys to select.

[root@mercury8] ~# growfs -s 94361839 da1s1


代码:


```
We strongly recommend you to make a backup before growing the Filesystem
```

```
Did you backup your data (Yes/No) ? Yes 注意大小写
```

```
new file systemsize is: 23590459 frags
```

```
Warning: 297836 sector(s) cannot be allocated.
```

```
growfs: 45929.7MB (94064000 sectors) block size 16384, fragment size 2048
```

```
using 250 cylinder groups of 183.72MB, 11758 blks, 23552 inodes.
```

```
super-block backups (for fsck -b #) at:   75627616, 76003872, 76380128, 76756384, 77132640, 77508896,
```

```
77885152, 78261408, 78637664, 79013920, 79390176, 79766432, 80142688, 80518944, 80895200, 81271456,
```

```
81647712, 82023968, 82400224, 82776480,   83152736, 83528992, 83905248, 84281504, 84657760, 85034016,
```

```
85410272, 85786528, 86162784, 86539040, 86915296, 87291552, 87667808, 88044064, 88420320, 88796576,
```

```
89172832, 89549088, 89925344, 90301600,   90677856, 91054112, 91430368, 91806624, 92182880, 92559136,
```

```
92935392, 93311648, 93687904
```

[root@mercury8] ~# fsck -y /data


代码:


```
** /dev/da1s1  ** Last Mounted on /data  ** Phase 1 - Check Blocks and Sizes  ** Phase 2 - Check Pathnames  **
```

```
Phase 3 - Check Connectivity  ** Phase 4 - Check Reference Counts  ** Phase 5 - Check Cyl groups  18062 files,
```

```
18276868 used, 4499090 free (474 frags, 562327 blocks, 0.0% fragmentation)    ***** FILE SYSTEM IS CLEAN *****
```

[root@mercury8] ~# mount /data


[root@mercury8] ~# df -h /data


Filesystem Size Used Avail Capacity Mounted on


/dev/da1s1 43G 35G 7.7G 82% /data


来源: [http://bbs.fyjy.net/showthread.php?t=4072](http://bbs.fyjy.net/showthread.php?t=4072)