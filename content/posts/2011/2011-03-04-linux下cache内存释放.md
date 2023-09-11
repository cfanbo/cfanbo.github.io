---
title: Linux下cache内存释放
author: admin
type: post
date: 2011-03-04T13:04:45+00:00
url: /archives/7952
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - Linux
 - vm
 - 内存释放

---
/proc是一个虚拟文件系统,我们可以通过对它的读写操作做为与kernel实体间进行通信的一种手段.也就是说可以通过修改/proc中的文 件,来对当前kernel的行为做出调整.那么我们可以通过调整/proc/sys/vm/drop_caches来释放内存.操作如下:

> [root@server test]# cat /proc/sys/vm/drop_caches
>

首先,/proc/sys /vm/drop_caches的值,默认为0

> [root@server test]# sync

手动执行sync命令(描述:sync 命令运行 sync 子例程。如果必须停止系统，则运行 sync 命令以确保文件系统的完整性。sync 命令将所有未写的系统缓冲区写到磁盘中，包含已修改的 i-node、已延迟的块 I/O 和读写映射文件)

> [root@server test]# echo 3 > /proc/sys/vm/drop_caches
> [root@server test]# cat /proc/sys/vm/drop_caches
> 3

将/proc/sys/vm/drop_caches值设为3

> [root@server test]# free -m
> total used free shared buffers cached
> Mem: 249 66 182 0 0 11
> -/+ buffers/cache: 55 194
> Swap: 511 0 511

再来运行free命令,发现现在的used为66MB,free为182MB,buffers为0MB,cached为11MB.那么有效的释放了 buffer和cache.

有关/proc/sys/vm/drop_caches的用法在下面进行了说明

> /proc/sys/vm/drop_caches (since Linux 2.6.16)
> Writing to this file causes the kernel to drop clean caches,
> dentries and inodes from memory, causing that memory to become
> free.
>
> To free pagecache, use echo 1 > /proc/sys/vm/drop_caches; to
> free dentrie