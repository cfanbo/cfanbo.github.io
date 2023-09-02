---
title: ipfw:getsockopt(IP_FW_GET):Protocol not available
author: admin
type: post
date: 2011-04-17T13:23:20+00:00
url: /archives/9312
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - IPFW

---
本想通过防火墙限制一下，谁知输入出现下面的错误：

>

> [root@Aaronwang ~]# ipfw show

ipfw: getsockopt(IP_FW_GET): Protocol not available
>

看来是我在编译内核的时候没有把IPFW编译进来，如果确信编译过了,那一定是没有reboot的问题了,我就是当时忘记reboot,才出现这个问题的.看来又要再编译一次内核了！上次内核编译是00:59:01，这次又是在半夜，看来我还真是个夜猫子！呵呵！

>

> [root@Aaronwang ~]# uname -a

FreeBSD Aaronwang 7.2-RELEASE-p6 FreeBSD 7.2-RELEASE-p6 #5: Thu Jan 14 00:59:01 CST 2010 root@Aaron wang:/usr/obj/usr/src/sys/Aaron.wang i386
>

编译ipfw要在/root/Aaron.wang里面加入下列内容：

>

> options IPFIREWALL

这个选项将 IPFW 作为内核的一部分来启用。

options IPFIREWALL_VERBOSE

这个选项将启用记录通过 IPFW 的匹配了包含 ‘log’ 关键字规则的每一个包的功能。

options IPFIREWALL_VERBOSE_LIMIT=5

以每项的方式， 限制通过 syslogd记录的包的个数。 如果在比较恶劣的环境下记录防火墙的活动可能会需要这个选项。它能够避免潜在的针对 syslog 的洪水式拒绝服务攻击。

options IPFIREWALL_DEFAULT_TO_ACCEPT

这个选项默认地允许所有的包通过防火墙， 如果您是第一次配置防火墙，使用这个选项将是一个不错的主意。

options IPDIVERT

这一选项启用 NAT 功能。
>

在/etc/rc.conf里面启用firewall。

>

> firewall_enable=”YES” # 激活firewall防火墙

firewall_script=”/etc/rc.firewall” # firewall防火墙的默认脚本

firewall_type=”/etc/ipfw.conf” # firewall自定义脚本

firewall_quiet=”NO” # 起用脚本时，是否显示规则信息。现在为“NO”假如你的防火墙脚本已经定型，那么就可以把这里设置成“YES”了。

firewall_logging_enable=”YES” # 启用firewall的log记录。
>

您还可以指定firewall_type为下列配置规则之一：
open ── 允许所有流量通过。
client ── 只保护本机。
simple ── 保护整个网络。
closed ── 完全禁止除回环设备之外的全部 IP 流量。
UNKNOWN ── 禁止加载防火墙规则。
filename ── 到防火墙规则文件的绝对路径。
在/etc/syslog.conf里面记录日志。

!ipfw

*.* /var/log/ipfw.log

ipfw的配置命令：ipfw [-N] 命令 [编号] 动作 [log(日志)] 协议 地址 [其它选项]
一些常用的规则：

>

> ipfw add 00005 deny tcp from any to any in tcpflags syn,fin # 这是过滤扫描包

ipfw add 10001 allow tcp from any to 192.168.1.1 80 in # 开放http服务。

ipfw add 10002 allow tcp from any to 192.168.1.1 21 in # 开放ftp服务。

ipfw add 10000 allow tcp from 1.2.3.4 to 192.168.1.1 22 in# 向固定IP开放SSH服务。

ipfw add 30000 allow icmp from any to any icmptypes 3

ipfw add 30001 allow icmp from any to any icmptypes 4

ipfw add 30002 allow icmp from any to any icmptypes 8 out

ipfw add 30003 allow icmp from any to any icmptypes 0 in

ipfw add 30004 allow icmp from any to any icmptypes 11 in

#允许ping和 traceroute
>

使用

```
ipfw -q -f flush       # Delete all rules
```

```
可以清除所有ipfw规则
```