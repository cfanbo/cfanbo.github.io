---
title: FreeBSD IPFW 防火墙的安装和设置
author: admin
type: post
date: 2011-04-17T12:17:27+00:00
url: /archives/9309
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - IPFW

---
IPFW本身是FreeBSD内置的，要使用IPFW设置防火墙需要重新编译FreeBSD内核。注意，因为在编译后IPFW默认拒绝所有网络服务，包括对系统本身都会拒绝，所以在配置过程中一定要小心谨慎。

内核编译方法请参考:**Step 1，对IPFW的一些基本参数进行配置：**

**#cd /sys/i386/conf**//如果没有这个目录，说明你的系统没有安装Ports服务，要记得装上。

**#cp GENERIC ./kernel_IPFW**用 vi 打开kernel_IPFW文件，在文件未尾加入以下个行：

> **options IPFIREWALL**//将包过滤部分代码编译进内核。

**options IPFIREWALL_VERBOSE**//启用通过Syslogd记录日志；如果没有指定这个选项，即使你在过滤规则中指定了记录包，也不会真的记录它们。

**options IPFIREWALL_VERBOSE_LIMIT=10**

 //限制通过Syslogd记录的每项包规则的记录条数。如果你受到了大量的攻击，想记录防火墙的活动，但又不想由于Syslog洪水一般的记录将你淹没，那么这个选项将会很有用。当使用了这条规则，当规则链中的某一项达到限制数值时，它所对应的日志将不再记录下来。

**options IPFIREWALL_DEFAULT_TO_ACCEPT** //注意，关键地方了，本句把默认的规则动作从“deny”改成“allow”了，作用是在默认状态下IPFW将会接受任何的数据。输入完成后，保存，并退出。 **Step 2**，编译系统内核：

> **#/usr/sbin/config kernel_IPFW**

**#cd ../compile/kernel_IPFW**

//注意，FreeBSD 4.X版本是../../compile/kernel_IPFW，而FreeBSD 5.X版本却是../compile/kernel_IPFW。

**#make****#make install**//开始编译内核。**Step 3，** 编辑 **/etc/rc.conf** 加入如下参数：

> **firewall_enable=“YES“** //激活Firewall防火墙

**firewall_script=“/etc/ipfw.conf“**

 //Firewall防火墙的默认脚本

**firewall_type=“open“** //Firewall 自定义脚本

**firewall_quiet=“NO“** //启用脚本时是否显示规则信息；假如你不再修改防火墙脚本，那么可以把这里设成“YES“。

**firewall_logging_enable=“YES“**编辑 **/etc/syslog.conf** 文件，在文件最后加入：

> **！ipfw*****.* /var/log/ipfw.log**//这行的作用是将IPFW的日志写到/var/log/ipfw.log/文件里。你可以为日志文件指定其它路径。最后，重启服务器。 重启之后，你就可以用SSH登录你的服务器了，之后你可以在/ **etc/ipfw.conf** 中添加过滤规则来防止入侵了。**#options IPFIREWALL_DEFAULT_TO_ACCEPT(编译内核)****这样的设置，防火墙会处于全封闭状态，需要自定义打开端口来实现功能。****# ee /etc/ipfw.conf**

**增加下面内容：**

**#!/bin/sh**

# DNS服务器与客户端的通讯端口都是udp的53号端口，因此我们只有开放自己与DNS服务器之间的53号端口进行通信即可，如果你知道自己的DNS服务器的ip地址可以把下面的any改成你的DNS地址。


\# DNS
ipfw add allow udp from me to any 53 out
ipfw add allow udp from any 53 to me in

# DHCP的服务器与客户端的通讯端口是udp 67、68端口，其工作原理这里不多做介绍，如果你知道自己的DHCP服务器地址可以把其中的any改成你的DHCP服务器地址，前面两条是正常情况下的规则，如果你的DHCP服务器不是很可靠，你可以加上下面注释掉的两条，当然一般情况下这两条可以不加。

\# DHCP
ipfw add allow udp from me 68 to any 67 out
ipfw add allow udp from any 67 to me 68 in
\# ipfw add allow udp from any 68 to 255.255.255.255 67 out
\# ipfw add allow udp from any 67 to 255.255.255.255 68 in

# 在创建与ICMP有关的规则时，只能指定ICMP数据包的type而不能指定它的code。

 # 允许接受一些ICMP types （不支持codes）# 允许双向的path-mtu
ipfw add allow icmp from any to any icmptypes 3
ipfw add allow icmp from any to any icmptypes 4

我需要考虑是否需要ping我的网络之外的主机或运行traceroute命令，由于二者都需要，并希望收到相应的应答，但我并不希望互联网上的所有用户都可以对我运行ping 或traceroute命令，因此，我需要添加下面的规则：

# 允许我对外部的主机运行ping命令，并得到相应的应答
ipfw add allow icmp from me to any icmptypes 8 out
ipfw add allow icmp from any to me icmptypes 0 in

# 允许我运行traceroute命令
ipfw add allow icmp from any to any icmptypes 11 in

# ICMP type 8是一个查询请求，ICMP type 0是对查询请求的应答。由于我只允许反复地发出请求并接受应答，从而我可以ping别人而别人不能ping我。

如果想让别人也能ping自己的话，则需要把上面的规则反一下
ipfw add allow icmp from any to me icmptype 8 in
ipfw add allow icmp from me to nay icmptype 0 out

# 在运行traceroute命令时，就会向外发出UDP数据包。如果希望能够获得所有应答信息，我还必须允许系统接受所有的CMP type 11数据包。

\# TCP类型服务
\# 只允许向外发送信息包
ipfw add check-state
ipfw add deny tcp from any to any in established
ipfw add allow tcp from any to any out setup keep-state



**# 允许ssh等tcp端口的服务
ipfw add allow tcp from any to me 21,22,80,3306 in
ipfw add allow tcp from me 22 to any out
# 由于web服务通过80端口进来，但是出去的数据是随机的，所以还得再加一条：

ipfw add allow tcp from me to any out**

**options IPFIREWALL_DEFAULT_TO_ACCEPT(编译内核)****这样的设置 ，防火墙处于全开放状态，需要自定义关闭端口来实现功能。****编辑/etc/ipfw.conf配置文件****封闭mysql数据库端口3306****ipfw add deny tcp from any to me 3360 in****######### TCP ##########**

**ipfw add 00001 deny log ip from any to any ipopt rr**

**ipfw add 00002 deny log ip from any to any ipopt ts**

**ipfw add 00003 deny log ip from any to any ipopt ssrr**

**ipfw add 00004 deny log ip from any to any ipopt lsrr**

**ipfw add 00005 deny tcp from any to any in tcpflags syn,fin**

**# 这5行是过滤各种扫描包**



**ipfw add 10001 allow tcp from any to 10.10.10.1 80 in** **# 向整个Internet开放http服务。**

**ipfw add 10002 allow tcp from any to 10.10.10.1 21 in****# 向整个Internet开放ftp服务。**

**ipfw add 10000 allow tcp from 1.2.3.4 to 10.10.10.1 22 in**

**# 向Internet的xx.xx.xx.xx这个IP开放SSH服务。也就是只信任这个IP的SSH登陆。**

**# 如果你登陆服务器的IP不固定，那么就要设为：****add 10000 allow tcp from any to 10.10.10.1 22 in**



**add 19997 check-state
add 19998 allow tcp from any to any out keep-state setup
add 19999 allow tcp from any to any out**

**#这三个组合起来是允许内部网络访问出去，如果想服务器自己不和Internet进行tcp连接出去，可以把19997和19998去掉。（不影响Internet对服务器的访问）**



**########## UDP ##########
add 20001 allow udp from any 53 to 10.10.10.1**

**# 允许其他DNS服务器的信息进入该服务器，因为自己要进行DNS解析嘛~****add 29999 allow udp from any to any out****# 允许自己的UDP包往外发送。**

**########## ICMP #########**

**add 30000 allow icmp from any to any icmptypes 3**

**add 30001 allow icmp from any to any icmptypes 4**

**add 30002 allow icmp from any to any icmptypes 8 out**

**add 30003 allow icmp from any to any icmptypes 0 in**

**add 30004 allow icmp from any to any icmptypes 11 in**

**#允许自己ping别人的服务器。也允许内部网络用router命令进行路由跟踪**

FreeBSD下IPFW防火墙的开启与关闭: