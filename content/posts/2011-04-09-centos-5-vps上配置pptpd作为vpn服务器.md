---
title: CentOS 5 VPS上配置pptpd作为VPN服务器［瑞豪开源］
author: admin
type: post
date: 2011-04-09T15:50:22+00:00
url: /archives/9165
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - pptpd
 - vpn

---
美国VPS的一大用途就是做为加密的VPN服务器，在国内连上这些VPN服务器就可以无限制访问互联网。常用的VPN服务器一般分两种，一种是SSL VPN，代表软件有openvpn，这个VPN软件有Windows下的客户端软件；另外一种是pptpd VPN，Windows自带这种VPN的客户端支持。本文记录了在CentOS 5 VPS下安装pptpd VPN服务器的过程。

## 内核支持

pptpd VPN需要内核支持mppe，我们的VPS自带的内核已经把mppe编译进去了，没有把mppe另外当作内核的模块。

## 软件安装

要安装pptpd VPN，ppp和iptables这两个软件是必须安装的，安装命令：

>

```
yum install -y ppp iptables
```

然后下载pptpd的rpm包：

> 32位
> 64位

要注意64位的系统要下载64位的rpm包，32位的系统要下载32位的rpm包，别搞错了

64位系统安装命令：

> `rpm -ivh pptpd*.x86_64.rpm`

32位系统安装命令：

>

```
rpm -ivh pptpd*.i386.rpm
```

编辑配置文件 /etc/ppp/options.pptpd 内容如下：

>

```
name pptpd
refuse-pap
refuse-chap
refuse-mschap
require-mschap-v2
require-mppe-128
proxyarp
lock
nobsdcomp
novj
novjccomp
nologfd
ms-dns 208.67.222.222
ms-dns 208.67.220.220
```

编辑配置文件 /etc/pptpd.conf 内容如下：

>

```
option /etc/ppp/options.pptpd
logwtmp
localip 192.168.92.1
remoteip 192.168.92.11-15
```

编辑配置文件 /etc/ppp/chap-secrets,配置用户名为johndoe，密码为password，内容如下：

>

```
johndoe pptpd password *
```

修改配置文件/etc/sysctl.conf中的相应内容如下：

>

```
net.ipv4.ip_forward = 1
```

‘配置iptables:

>

```
iptables -t nat -A POSTROUTING -o eth0 -s 192.168.92.0/24 -j MASQUERADE
iptables -I FORWARD -p tcp --syn -i ppp+ -j TCPMSS --set-mss 1356
/etc/init.d/iptables save
/etc/init.d/iptables restart
```

设置iptables和pptpd开机自动启动：

>

```
chkconfig pptpd on
chkconfig iptables on
```

然后运行reboot重新启动即可.

来源： [http://rashost.com/blog/centos5-pptpd-vpn](http://rashost.com/blog/centos5-pptpd-vpn)