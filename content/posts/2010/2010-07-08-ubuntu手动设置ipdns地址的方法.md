---
title: ubuntu手动设置IP/DNS地址的方法
author: admin
type: post
date: 2010-07-08T03:26:52+00:00
url: /archives/4508
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Ubuntu

---
Ubuntu的网络参数保存在文件 /etc/network/interfaces中，默认设置使用dhcp，内容如下：

\# The primary network interface
auto eth0
iface eth0 inet dhcp

设置静态ip的方法如下：
1） 编辑 /etc/network/interfaces
1.1）将dhcp 一行屏蔽
\# The primary network interface
auto eth0
#iface eth0 inet dhcp
1.2）添加和静态ip有关的参数


\# The primary network interface
iface eth0 inet static
address 192.168.0.10
netmask 255.255.255.0
gateway 192.168.0.1

2）编辑 /etc/resolv.conf，设置dns
nameserver 202.96.134.133
nameserver 202.106.0.20

3）执行下面两个命令，启用新设置
 **$sudo ifdown eth0
$sudo ifup eth0**

网络重启可以用下面的指今：
**#sudo /etc/init.d/networking restart**