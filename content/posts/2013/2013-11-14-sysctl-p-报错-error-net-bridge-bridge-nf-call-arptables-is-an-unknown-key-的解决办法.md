---
title: 'sysctl -P 报错 error: “net.bridge.bridge-nf-call-arptables” is an unknown key 的解决办法'
author: admin
type: post
date: 2013-11-14T03:22:21+00:00
url: /archives/14721
categories:
 - 服务器
tags:
 - openvz

---
今天在安装openvz的时候(安装教程： [http://blog.haohtml.com/archives/14724](/archives/14724))，修改完内核参数后，执行

[shell]sysctl -P[/shell]

后，提示

[shell]net.ipv4.ip_forward = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept\_source\_route = 0
kernel.sysrq = 1
kernel.core\_uses\_pid = 1
net.ipv4.tcp_syncookies = 1
error: "net.bridge.bridge-nf-call-ip6tables" is an unknown key
error: "net.bridge.bridge-nf-call-iptables" is an unknown key
error: "net.bridge.bridge-nf-call-arptables" is an unknown key
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.shmmax = 68719476736
kernel.shmall = 4294967296[/shell]

错误．解决方法：

[shell]modprobe bridge
lsmod|grep bridge[/shell]