---
title: 在FreeBSD上架VPN笔记
author: admin
type: post
date: 2011-05-18T06:22:16+00:00
url: /archives/9458
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - vpn

---
摘自:

在 Freebsd 上用 mpd5 构建 PPTP VPN

## 安装 MPD5

```
cd /usr/ports/net/mpd5
make install clean
```

在 /etc/rc.conf 中启用 mpd5 添加如下行


```
mpd_enable="YES"
```

## 配置 mpd pptp VPN

复制默认的 mpd.conf 配置文件


```
cd /usr/local/etc/mpd5/
cp mpd.conf.sample mpd.conf
```

修改 mpd.conf 文件中的 startup: default: pptp_server: 三块，其它的不要理睬，放在里面不要删除，因为可以通过 default: 标签来调用需要执行的模块，所以不受影响。

以下是这三部分的代码，需要修改的地方见我的中文解释。


```
startup:
        # configure mpd users
        set user admin password ### 设置 mpd 的访问帐号及密码，通过 telnet 或 web 访问时需要此帐号
        #set user foo1 bar1
        # configure the console
        set console self 127.0.0.1 5005
        set console open
        # configure the web server
        set web self 0.0.0.0 5006
        set web open

#
# Default configuration is "dialup"

default:
        #load dialup
        load pptp_server ### 默认调用 pptp_server 模块

pptp_server:
#
# Mpd as a PPTP server compatible with Microsoft Dial-Up Networking clients.
#
# Suppose you have a private Office LAN numbered 192.168.1.0/24 and the
# machine running mpd is at 192.168.1.1, and also has an externally visible
# IP address of 1.2.3.4.
#
# We want to allow a client to connect to 1.2.3.4 from out on the Internet
# via PPTP.  We will assign that client the address 192.168.1.50 and proxy-ARP
# for that address, so the virtual PPP link will be numbered 192.168.1.1 local
# and 192.168.1.50 remote.  From the client machine's perspective, it will
# appear as if it is actually on the 192.168.1.0/24 network, even though in
# reality it is somewhere far away out on the Internet.
#
# Our DNS server is at 192.168.1.3 and our NBNS (WINS server) is at 192.168.1.4.
# If you don't have an NBNS server, leave that line out.
#

# Define dynamic IP address pool.
        set ippool add pool1 192.168.1.50 192.168.1.99

# Create clonable bundle template named B
        create bundle template B
        set iface enable proxy-arp
        set iface idle 1800
        set iface enable tcpmssfix
        set ipcp yes vjcomp
# Specify IP address pool for dynamic assigment.
        set ipcp ranges 192.168.1.1/32 ippool pool1
        set ipcp dns 8.8.8.8  ### 设置 dns
        #set ipcp nbns 192.168.1.4 ###如果你用不到 wins 的话，可以注释掉这块，
# The five lines below enable Microsoft Point-to-Point encryption
# (MPPE) using the ng_mppc(8) netgraph node type.
        set bundle enable compression
        set ccp yes mppc
        set mppc yes e40
        set mppc yes e128
        set mppc yes stateless
# Create clonable link template named L
        create link template L pptp
# Set bundle template to use
        set link action bundle B
# Multilink adds some overhead, but gives full 1500 MTU.
        set link enable multilink
        set link yes acfcomp protocomp
        set link no pap chap eap
        set link enable chap

# We can use use RADIUS authentication/accounting by including
# another config section with label 'radius'.
#       load radius
        set link keep-alive 10 60
# We reducing link mtu to avoid GRE packet fragmentation.
        set link mtu 1460
# Configure PPTP
        set pptp self 202.101.8.18 ###设置 pptp 的监听 ip 地址，也就是你的网卡的 IP 地址
# Allow to accept calls
        set link enable incoming
```

好了，就这么简单。


## 启动 mpd5

```
 /usr/local/etc/rc.d/mpd5 start
```

检查 mpd5 是否已经启动


```
netstat -a
```

可以看到类似于这样的输出信息


```
tcp4       0      0 vpn.server..pptp  *.*                    LISTEN
```

说明 pptp 已正常启动


## 添加 VPN 帐号

创建 /usr/local/etc/mpd5/mpd.secret 文件，输入用户名及密码，一行一个


如


```
vpnaccount1  password1
vpnaccount2  password2
riku         password3
```

然后就可以在 windows 下尝试登录 vpn 服务器了


## 启用包转发

以上配置好后，但只能访问内部网络，而不访问外网，所以要让服务器启用包转发。我用 ipfw 来提供此功能。


在 /etc/rc.conf 中加入以下行


```
gateway_enable="YES"
firewall_enable="YES"
firewall_type="OPEN"
firewall_logging_enable="YES"
natd_enable="YES"
natd_interface="em0" // em0 为网卡型号，你可以用 ifconfig 来检查你的网卡型号
```

编辑 ifpw 的规则文件 /etc/ipfw.rules


```
ipfw add allow all from any to any
ipfw add divert natd ip from any to any via em0
```

最后重新启动


当然如果你不想重启的话，也可以通过以下命令来启用包转发。


```
sysctl net.inet.ip.forwarding=1
/etc/rc.d/ipfw start
```

好了，配置完成。


## 高级：使用主机系统帐号登录

在 mpd.conf 中的 # load radius 后添加两行


```
        set auth disable internal  ### 禁止使用 mpd.secert 文件作为帐户认证
        set auth enable system-auth ### 添加系统认证方式
```

修改 /etc/login.conf ，把帐号加密方式改为 nth


```
        :passwd_format=nth:\
        #:passwd_format=md5:\
```

重建 login.conf 数据库


```
cap_mkdb /etc/login.conf
```

最后添加新帐户


```
adduser
```

# 用 mpd5 作为 pptp client 连接 VPN

这块也很简单，只要改一下默认配置中的几行，见下面代码中的中文注释。


另外，上面的default: 标签中要加入 load pptp_client 这行，以便重启 mpd 时加载 pptp client 模块。


```
pptp_client:
#
# PPTP client: only outgoing calls, auto reconnect,
# ipcp-negotiated address, one-sided authentication,
# default route points on ISP's end
#

        create bundle static B1
        set iface route default
        set ipcp ranges 0.0.0.0/0 0.0.0.0/0

        create link static L1 pptp
        set link action bundle B1
        set auth authname riku  ###VPN 帐号
        set auth password password  ###  VPN 密码
        set link max-redial 0
        set link mtu 1460
        set link keep-alive 20 75
        set pptp peer 208.1.2.3 ### VPN 服务器的 ip 地址
        set pptp disable windowing
        open
```

重启 mpd5


```
/usr/local/etc/rc.d/mpd5 restart
```

最后还需要重新设一下路由。


# 参考

mpd5.5 手册 [http://mpd.sourceforge.net/doc5/mpd.html](http://mpd.sourceforge.net/doc5/mpd.html "http://mpd.sourceforge.net/doc5/mpd.html") ipfw [http://www.freebsd.org/doc/zh_CN/books/handbook/firewalls-ipfw.html](http://www.freebsd.org/doc/zh_CN/books/handbook/firewalls-ipfw.html "http://www.freebsd.org/doc/zh_CN/books/handbook/firewalls-ipfw.html")