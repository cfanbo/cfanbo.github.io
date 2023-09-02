---
title: FreeBSD利用poptop架设vpn指南
author: admin
type: post
date: 2010-07-11T14:44:01+00:00
url: /archives/4576
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - poptop
 - vpn

---
安装poptop
cd /usr/ports/net/poptop
make install clean
===> SECURITY REPORT:
This port has installed the following files which may act as network
servers and may therefore pose a remote security risk to the system.
/usr/local/sbin/pptpd

This port has installed the following startup scripts which may cause
these network services to be started at boot time.
/usr/local/etc/rc.d/pptpd

cd /usr/local/etc
cp pptpd.conf.sample pptpd.conf
vi pptpd.conf
cd /usr/ports/net/poptop
make install clean

三、设置/usr/local/etc/pptpd.conf
cd /usr/local/etc
cp pptpd.conf.sample pptpd.conf
加入以下：
###############################

debug
nobsdcomp
proxyarp
localip 192.168.100.61
remoteip 192.168.100.62-70
pidfile /var/run/pptpd.pid
+chapms-v2
mppe-40
mppe-128
mppe-stateless

###############################

四、设置/etc/ppp/ppp.conf
cd /etc/ppp
加入以下：
###############################

loop:
set timeout 0
set log phase chat connect lcp ipcp command
set device localhost:pptp
set dial
set login
\# Server (local) IP address, Range for Clients, and Netmask
\# if you want to use NAT use private IP addresses
set ifaddr 192.168.100.61 192.168.100.62-192.168.100.70 255.255.255.0
add default HISADDR
set server /tmp/loop “” 0177

loop-in:
set timeout 0
set log phase lcp ipcp command
allow mode direct

pptp:
load loop
disable pap
\# Authenticate against /etc/passwd
enable passwdauth
disable ipv6cp
enable proxy
accept dns
enable MSChapV2
enable mppe
disable deflate pred1
deny deflate pred1
set dns 221.228.255.1
set device !/etc/ppp/secure

###############################

五、设置VPN登陆用户和密码/etc/ppp/ppp.secret

USERNAME PASSWORD

六、设置/etc/sysctl.conf
加入以下：
net.inet.ip.forwarding=1
执行
sysctl net.inet.ip.forwarding=1

七、设置/etc/rc.conf
加入以下：
arpproxy_all=”YES”
gateway_enable=”YES”
执行
cd /etc
sh rc

八、cd /usr/local/etc/rc.d
cp pptpd.sh.sample pptpd.sh
./pptpd.sh start

到现在VPN服务器就启动完毕了，就可以用windows登陆
如果log里面出现configuration label not found，检查ppp.conf里面，除了loop，pptp什么的以外，其他的各行前面都是tab而不是空格