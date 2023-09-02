---
title: Centos下vpn（pptpd）的部署
author: admin
type: post
date: 2011-04-09T10:44:34+00:00
url: /archives/9134
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - pptpd
 - vpn

---
现在大多数VPS服务器都位于国外，因此PPTP服务器可以用来搭建一个比较实用的代理服务器。PPTP服务需要用到特定的Linux内核技术，因此绝大多数OpenVZ架构的VPS都不能配置PPTP（可以用OPENVPN代替），但几乎所有的XEN或KVM架构的VPS都能正常安装。本文将以XEN架构的CentOS系统下安装PPTP服务为例进行讲解。

### 1、准备环境

PPTPD要求Linux内核支持mppe，一般来说CentOS安装时已经包含了；下面安装ppp与iptables：

```
yum install perl ppp iptables
```

### 2、安装PPTPD

对于32位CentOS，执行

```
wget http://acelnmp.googlecode.com/files/pptpd-1.3.4-1.rhel5.1.i386.rpm
rpm -ivh pptpd-1.3.4-1.rhel5.1.i386.rpm
```

对于64位CentOS，执行

```
wget http://acelnmp.googlecode.com/files/pptpd-1.3.4-1.rhel5.1.x86_64.rpm
rpm -ivh pptpd-1.3.4-1.rhel5.1.x86_64.rpm
```

### 3、修改配置

**编辑PPTP配置文件 /etc/ppp/options.pptpd 如内容下：**

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
idle 2592000
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

**编辑配置文件 /etc/pptpd.conf ，内容如下：**

```
option /etc/ppp/options.pptpd
logwtmp
localip 192.168.254.1
remoteip 192.168.254.100-254
```

* 其中localip与remoteip定义了客户端连接VPN服务器后被分配到的内网IP地址，可根据需要自己修改。

**现在对用户认证文件 /etc/ppp/chap-secrets 进行配置，内容如下：**

```
testuser pptpd testpwd *
```

* testuser、testpwd对应修改为自己希望的VPN登录用户名和密码

**将 /etc/sysctl.conf 文件中net.ipv4.ip_forward设置为 1 （如果没有，则按照格式新建一行）：**

```
net.ipv4.ip_forward = 1
```

保存退出。执行

```
/sbin/sysctl -p
```

使之生效。



### 4、设置iptables转发

```
/etc/init.d/iptables start
/sbin/iptables -t nat -A POSTROUTING -o eth0 -s 192.168.254.0/24 -j MASQUERADE
/etc/init.d/iptables save
/etc/init.d/iptables restart
```

* 注意，上面的192.168.254.0应该与之前设置的网段对应。



### 5、设置开机启动

```
chkconfig pptpd on
chkconfig iptables on
```

重启计算机即可进行连接，并且能够正常上网。

如果重启服务器后，无法连接VPN，首先检查服务器的PPTP服务1723端口是否已打开（注意设置防火墙允许此端口）；如果可以连接VPN，但是无法正常上网，则检查iptables是否正常转发。

**6、VPN客户端配置（windows）。**

1、建立VPN连接，鼠标右键单击桌面的“网上邻居”图标，选择快捷菜单中的“属性”命令。 双击“新建连接向导”图标。 在出现的“欢迎使用新建连接向导”对话框中单击“下一步”按钮,选择＂连接到我的工作场所的网络＂如图所示:

[![](http://blog.haohtml.com/wp-content/uploads/2011/04/vpn_netlink.jpg)](http://blog.haohtml.com/wp-content/uploads/2011/04/vpn_netlink.jpg)

2、点击下一步，选择＂虚拟专用网络连接＂：

[![](http://blog.haohtml.com/wp-content/uploads/2011/04/vpn_netlink-2.jpg)](http://blog.haohtml.com/wp-content/uploads/2011/04/vpn_netlink-2.jpg)

3、输入连接的名称

[![](http://blog.haohtml.com/wp-content/uploads/2011/04/vpn_netlink-3.jpg)](http://blog.haohtml.com/wp-content/uploads/2011/04/vpn_netlink-3.jpg)

4、输入VPN服务器的域名或IP地址

[![](http://blog.haohtml.com/wp-content/uploads/2011/04/vpn_netlink-4.jpg)](http://blog.haohtml.com/wp-content/uploads/2011/04/vpn_netlink-4.jpg) 5、在最后一步，选择在桌面创建一个快捷方式

**六、拨号登录。**

在桌面上双击“VPN连接图标”图标，在出现的连接对话框中输入登录VPN服务器的用户名和密码，然后单击“连接”按钮。

**相关教程：**

另外一篇文章： [http://rashost.com/blog/centos5-pptpd-vpn](http://rashost.com/blog/centos5-pptpd-vpn)