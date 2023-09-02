---
title: Freebsd解决ARP欺骗问题
author: admin
type: post
date: 2011-10-31T05:54:34+00:00
url: /archives/11899
IM_contentdowned:
 - 1
categories:
 - 服务器

---
**1.不安装软件的方法。**
首先要重启，确保你拿到的网关地址是正确的。
步骤如下：

> ifconfig

显示类似如下内容

> bge0: flags=8843 mtu 1500
> options=1b
> inet 192.168.0.5 netmask 0xffffffc0 broadcast 192.168.0.1
> ether 00:17:08:2a:13:88
> media: Ethernet autoselect (100baseTX )
> status: active
> plip0: flags=108810 mtu 1500
> lo0: flags=8049 mtu 16384
> inet 127.0.0.1 netmask 0xff000000

我们把网关的信息存到一个文件里。

> echo 192.168.0.1 00:17:08:2a:13:88 > /etc/ipmac

接着使用crontab -e编辑系统定时排程（计划任务）让它按照设定时间循环执行

> \*/5 \* \* \* * /usr/sbin/arp -f /etc/ipmac

这样就每5分钟更新一次网关MAC地址，保证正确。


注意：这里有一个隐患，那就是如果网关设备更换，也就是网关的MAC地址变了就会发生网络不通的现象了。因此就要去机房修改ipmac文件，将新网关MAC改进去，所以我的建议是，先备份ipmac，但是不要马上定时更新，而是等发现有ARP病毒了，再更新，等病毒消除了，就停止更新。确保网络连接正常。

**2.安装防ARP的保护软件**

> ****cd /usr/ports/security/ipguard/
> make install

安装完之后会建立/etc/ethers文件来保护本机，抵御arp欺骗、攻击。
启动ipguard.

> cd /usr/local/etc/rc.d
> mv ipguard.sh.sample ipguard.sh
> /usr/local/etc/rc.d/ipguard.sh start

ipguard用法详解

> ipguard – tool designed to protect LAN IP adress space by ARP spoofing.
>
> ipguard listens network for ARP packets. All permitted MAC/IP pairs
> listed in ‘ethers’ file. If it recieves one with MAC/IP pair, which is
> not listed in ‘ethers’ file, it will send ARP reply with configured
> fake address. This will prevent not permitted host to work properly in
> this ethernet segment. Especially Windows(TM) hosts.

[**EXAMPLES**](http://www.freebsd.org/cgi/man.cgi?query=ipguard&apropos=0&sektion=0&manpath=FreeBSD+7.0-RELEASE+and+Ports&format=html#end)
Normal method, duplex, autoupdate /etc/ethers every 5 min and send 2
fake replies:
**ipguard** **-x** **-u** **300** **-n** **2** **fxp0**

Read-only mode and remember last 100 not listed in \`ethers’ MACs. Use-
ful for initial MAC/IP pairs collect:
**ipguard** **-r** **-b** **100** **-f** **./empty_file** **rl0**

Do not go to background and be more verbose, with test ethers file:
**ipguard** **-dv** **-f** **/tmp/ethers** **my1**

[**TIPS**](http://www.freebsd.org/cgi/man.cgi?query=ipguard&apropos=0&sektion=0&manpath=FreeBSD+7.0-RELEASE+and+Ports&format=html#end)
You must have read permission on /dev/bpf* if you want to start ipguard
in read-only mode and read/write permission for full functional if
you’re not root.

First MAC/IP pair in list always taken from listening interface, so you
can’t occasionally block yourself.

[**BUGS**](http://www.freebsd.org/cgi/man.cgi?query=ipguard&apropos=0&sektion=0&manpath=FreeBSD+7.0-RELEASE+and+Ports&format=html#end)
ipguard will not prevent changing MAC address along with IP by client.

Linux send weird ARP packet when enters net. ipguard did’nt handle it.

Signals like HUP or TERM works only on new received arp packet.