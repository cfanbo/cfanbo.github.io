---
title: Linux系统设置–ulimit
author: admin
type: post
date: 2011-06-16T11:36:33+00:00
url: /archives/9883
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ulimit

---
**功能说明：**控制shell程序的资源。

**语　　法：**ulimit \[-aHS\]\[-c \]\[-d <数据节区大小>\]\[-f <文件大小>\]\[-m <内存大小>\]\[-n <文件数目>\]\[-p <缓冲区大小>\]\[-s <堆叠大小>\]\[-t \]\[-u <程序数目>\][-v <虚拟内存大小>]

**补充说明：**ulimit为shell内建指令，可用来控制shell执行程序的资源。

**参　　数：**
-a 　显示目前资源限制的设定。
-c 　设定core文件的最大值，单位为区块。
-d <数据节区大小> 　程序数据节区的最大值，单位为KB。
-f <文件大小> 　shell所能建立的最大文件，单位为区块。
-H 　设定资源的硬性限制，也就是管理员所设下的限制。
-m <内存大小> 　指定可使用内存的上限，单位为KB。
-n <文件数目> 　指定同一时间最多可开启的文件数。
-p <缓冲区大小> 　指定管道缓冲区的大小，单位512字节。
-s <堆叠大小> 　指定堆叠的上限，单位为KB。
-S 　设定资源的弹性限制。
-t 　指定CPU使用时间的上限，单位为秒。
-u <程序数目> 　用户最多可开启的程序数目。
-v <虚拟内存大小> 　指定可使用的虚拟内存上限，单位为KB。

**方法：**
1、修改/etc/sysctl.conf
vi /etc/sysctl.conf
\# 确认包含下面内容

```
#------Begin Update for ASPire-----
fs.file-max = 32768
kernel.msgmni = 1024
kernel.sem = 1000 32000 32 512
kernel.shmmax = 2147483648
net.ipv4.tcp_fin_timeout = 10
net.ipv4.tcp_max_syn_backlog = 4096
#------End Update for ASPire-------
```

让配置立即生效

```
sysctl -p
```

2、修改/etc/security/limits.conf

\# 确认包含下面的内容：

```
* soft nofile 8192
* hard nofile 8192
```

修改后，用  ulimit -Hn  和 ulimit -Sn 确认修改已生效

使用命令

```
ulimit -HSn 65536
```

可以立即生效.