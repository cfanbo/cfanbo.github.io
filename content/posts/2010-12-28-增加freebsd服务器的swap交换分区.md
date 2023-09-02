---
title: 增加FreeBSD服务器的swap交换分区
author: admin
type: post
date: 2010-12-28T02:45:07+00:00
url: /archives/7321
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - 交换分区

---
 **** ******晚上有客户反映服务器无法访问了，我好不容易蹭了附近邻居的一个无线网络，连上服务器后发现了很多异常链接，swap交换空间占用99%左右，日志中发现如下记录**

Jul 27 23:52:19 freebsd1 kernel: pid 49901 (httpd), uid 1002, was killed: out of swap space

立即重启了apache后，swapinfo显示占用情况很快从5%迅速上升到64%直到99%

 **在 FreeBSD 中创建交换文件**

1. 确认您的内核配置包含虚拟磁盘(Memory disk)驱动 (md(4))。它在 GENERIC 内核中是默认的。

```
device   md   # Memory "disks"
```

2. 创建一个交换文件 **64M**(/usr/swap0)：

```
# dd if=/dev/zero of=/usr/swap0 bs=1024k count=64
```

3. 赋予它(/usr/swap0)一个适当的权限：

```
# chmod 0600 /usr/swap0
```

4. 在 /etc/rc.conf 中启用交换文件：

```
swapfile="/usr/swap0"   # Set to name of swapfile if aux swapfile desired.
```

5. 通过重新启动机器或下面的命令使交换文件立刻生效：

```
# mdconfig -a -t vnode -f /usr/swap0 -u 0 && swapon /dev/md0
```


成功加载新交换分区后

freebsd1# **swapinfo**

> Device          1K-blocks     Used    Avail Capacity
>
> /dev/ad1s1b       2097152   494016  1603136    24%
>
> /dev/md0          2097152   426576  1670576    20%
>
> Total             4194304   920592  3273712    22%

问题搞定！！！

随即尝试增加swap分区大小解决问题;

摘自: