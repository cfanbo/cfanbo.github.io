---
title: FreeBSD 单网卡绑定多个IP
author: admin
type: post
date: 2010-10-29T07:20:15+00:00
url: /archives/6439
IM_contentdowned:
 - 1
categories:
 - 服务器

---
假设网卡lnc0原IP地址为192.168.0.2，现在为它绑定另一个IP：

> \# ifconfig lnc0 192.168.0.3 netmask 255.255.255.255 alias

**解释：**

如果别名IP地址和网卡原IP地址在同一个子网上，就需要设置掩码为255.255.255.255

如果位于不同的子网，就直接使用相应子网的正常网络掩码

从TCP/IP的角度来看，这样做意味着什么呢？

网络掩码的所有位都设置成1，就会保证ICP/IP栈这样来看待包：

只要包的目标地址匹配所有位，就把该包看成本地子网上的包；它创建了只有一个地址的“子网”。

所有发送给该地址的包以及该地址接受的包都会发送给路由器，而不会发送到LAN上。

如果多个别名使用了同一个网络掩码，这些别名的广播地址也应该相同，而这样却导致了TCP/IP栈的混乱。

使用全1的网络掩码，才能骗过ifconfig，让该命令允许单个接口卡上有多个IP地址。

要在/etc/rc.conf中设置别名，应该使用 ifconfig\_xxx#\_alias# 关键字，该关键字的使用形式类似于 ifconfig_xxx#：

> ifconfig_lnc0=”inet 192.168.0.2 netmask 255.255.255.0″
> ifconfig\_lnc0\_alias0=”inet 192.168.0.3 netmask 255.255.255.255″
> ifconfig\_lnc0\_alias1=”inet 192.168.1.2 netmask 255.255.255.0″
> ifconfig\_lnc0\_alias2=”inet 192.168.1.3 netmask 255.255.255.255″