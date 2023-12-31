---
title: Linux命令：ifconfig
author: admin
type: post
date: 2010-07-08T08:37:10+00:00
url: /archives/4545
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - linux.ifconfig

---
## Linux命令：ifconfig

**功能说明**：显示或设置网络设备

 **英文说明：**network **i**nter**f**aces **config**uring

### 语法

**语　法**：ifconfig \[网络设备\]\[down up -allmulti -arp -promisc\]\[add<地址>\]\[del<地址>\]\[<硬件地址>\] \[media<网络媒介类型>\]\[mem_start<内存地址>\]\[metric<数目>\]\[mtu<字节>\]\[netmask<子网掩码>\]\[tunnel<地址>\]\[-broadcast<地址>\] [-pointopoint<地址>]

### 补充说明

**补充说明**：ifconfig可设置网络设备的状态，或是显示目前的设置。

### 参数

[网络设备] 网络设备的名称。

down 关闭指定的网络设备。

up 启动指定的网络设备。

-arp 打开或关闭指定接口上使用的ARP协议。前面加上一个负号用于关闭该选项。

-allmuti 关闭或启动指定接口的无区别模式。前面加上一个负号用于关闭该选项。

-promisc 关闭或启动指定网络设备的promiscuous模式。前面加上一个负号用于关闭该选项。

add<地址> 设置网络设备IPv6的IP地址。

del<地址> 删除网络设备IPv6的IP地址。

media<网络媒介类型> 设置网络设备的媒介类型。

mem_start<内存地址> 设置网络设备在主内存所占用的起始地址。

metric<数目> 指定在计算数据包的转送次数时，所要加上的数目。

mtu<字节> 设置网络设备的MTU。

netmask<子网掩码> 设置网络设备的子网掩码。

tunnel<地址> 建立IPv4与IPv6之间的隧道通信地址。

-broadcast<地址> 将要送往指定地址的数据包当成广播数据包来处理。

-pointopoint<地址> 与指定地址的网络设备建立直接连线，此模式具有保密功能。

## Linux中对网卡进行编辑的命令

无论是Linux 自动安装还是我们手工安装，Linux 都会向你询问有关网络的问题并配置相关的软件。这个用于配置网卡的基本命令就是ifconfig。

在执行ifconfig 命令后，系统将在内核表中设置必要的参数，这样Linux 就知道如何与网络上的网卡通信。ifconfig 命令有以下两种格式：

※ifconfig [interface]

※ifconfig interface [aftype] option | address …

ifconfig 的第一种格式（或使用不带任何参数的ifconfig 命令）可以用来查看当前系统的网络配置情况。

在刚刚安装完系统之后，实际上是在没有网卡或者网络连接的情况下使用Linux，但通过ifconfig 可以使用回绕方式工作，使计算机认为自己工作在网络上。

现在我们运行一下ifconfig 命令，不带参数的ifconfig 命令可以显示当前启动的网络接口，其输出结果为：

[root@machine1 /sbin]#ifconfig

eth0 Link encap:Ethernet HWaddr 52:54:AB:DD:6F:61

inet addr:210.34.6.89 Bcast:210.34.6.127 Mask:255.255.255.128

UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1

RX packets:46299 errors:0 dropped:0 overruns:0 frame:189

TX packets:3057 errors:0 dropped:0 overruns:0 carrier:0

collisions:0 txqueuelen:100

Interrupt:5 Base address:0xece0

lo Link encap:Local Loopback

inet addr:127.0.0.1 Mask:255.0.0.0

UP LOOPBACK RUNNING MTU:3924 Metric:1

RX packets:44 errors:0 dropped:0 overruns:0 frame:0

TX packets:44 errors:0 dropped:0 overruns:0 carrier:0

collisions:0 txqueuelen:0

其中以eth0 为首的部分是本机的以太网卡配置参数，的设这里显示了网卡的设备名/dev/eth0 和硬件的MAC 地址52:54:AB:DD:6F:61，MAC 地址是生产厂家定的，每个网卡拥有的唯一地址。

不过我们可以手工改动网卡的MAC 地址，只要我们在/etc/rc.d/init.d/中的network 中加入：

ifconfig eth0 hw ether xx:xx:xx:xx:xx:xx

Jiania 解说 注:

eth0，eth1,eth2,代表网卡一，网卡二，网卡三

hw 代表hardware 硬件意思

ether 代表ethernet 以太网的意思

然后重启，此时再用ifconfig 命令查看一下，我们就会发现网卡的MAC 地址已经变成xx:xx:xx:xx:xx:xx了。

## ifconfig配置网卡

配置网卡的IP地址

ifconfig eth0 192.168.0.1 netmask 255.255.255.0

在eth0上配置上192.168.0.1 的IP地址及24位掩码。若想再在eth0上在配置一个192.168.1.1/24 的IP地址怎么办？用下面的命令

ifconfig eth0：0 192.168.1.1 netmask 255.255.255.0

这时再用ifconifg命令查看，就可以看到两个网卡的信息了，分别为：eth0和eth0：0.若还想再增加IP，那网卡的命名就接着是：eth0：1、eth0：2……想要几个就填几个。ok！

配置网卡的硬件地址

ifconfig eth0 hw ether xx：xx：xx：xx：xx：xx就将网卡的硬件地址更改了，此时你就可以骗过局域网内的IP地址邦定了。

将网卡禁用

ifconfig eth0 down

将网卡启用

ifconfig eth0 up

ifconfig 命令的功能很强大，还可以设置网卡的MTU，混杂模式等。就不一一介绍了，用时间可以自己研究一下。