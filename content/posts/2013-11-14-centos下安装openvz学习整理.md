---
title: centos下安装openvz(转载整理)
author: admin
type: post
date: 2013-11-14T05:42:38+00:00
url: /archives/14724
categories:
 - 系统架构
 - 服务器
tags:
 - 虚拟化
 - openvz
 - vps

---
经常有人问到 OpenVZ 和 Xen 哪个好，事实上 OpenVZ 和 Xen 不是同一层面的技术，OpenVZ 是操作系统层面（ [Operating system-level virtualization](http://en.wikipedia.org/wiki/Operating_system-level_virtualization)）的虚拟产品，和 FreeBSD Jail, Solaris Zone, Linux-VServer 等类似；而 Xen 和 VMware, KVM, Hyper-V 等产品站在同一阵营。OpenVZ VPS 实际上提供的是一个虚拟环境（Virtual Environment/VE），也叫容器（Container）；而 Xen VPS 提供的是基于 Hypervisor 的虚拟机（Virtual Machine），这是本质上的不同，现在大家已经习惯用 VPS 这个名字把这两种不同的产品和技术混为一谈了。比起 Xen 专注于企业虚拟化和云计算领域，OpenVZ 最大的应用可能就在低端 VPS 市场，有无数的 VPS 服务商都使用 OpenVZ 提供 [10美元以下的 VPS](http://www.vpsee.com/category/host/) 产品。了解一下 OpenVZ 的安装和配置也会对使用 OpenVZ VPS 有所帮助，以下的安装和配置操作在 VPSee 的一台空闲 PC 和 CentOS 5.5 上完成。对 Xen 和 KVM 感兴趣的童鞋请看： [在 CentOS 上安装和配置 Xen](http://www.vpsee.com/2009/07/install-xen-on-centos/) (或)和 [在 CentOS 上安装和配置 KVM](http://www.vpsee.com/2010/07/install-kvm-on-centos/).

## 安装 OpenVZ

首先加入 openvz 源、升级系统、安装 openvz 内核和 vzctl, vzquota 等工具：

```
# cd /etc/yum.repos.d
# wget http://download.openvz.org/openvz.repo
# rpm --import http://download.openvz.org/RPM-GPG-Key-OpenVZ
# yum update

# yum install vzkernel
# yum install vzctl vzquota
```

## 调整内核参数

为了能让 VE/VPS 访问外部网络，我们必须启动 ip forwarding；如果内核出错或者运行很慢，我们希望能用特殊按键 dump 一些信息到控制台并结合 log 排错，所以建议打开 kernel.sysrq：

```
# vi /etc/sysctl.conf
...
net.ipv4.ip_forward = 1
kernel.sysrq = 1
...
```

为了减少麻烦最好关闭 selinux，selinux 带来的麻烦往往比得到的好处多：

```
# vi /etc/sysconfig/selinux
...
SELINUX=disabled
...
```

检查 vz 服务是否自动启动，并重启机器进入 openvz 内核：

```
# chkconfig --list vz
vz             	0:off	1:off	2:on	3:on	4:on	5:on	6:off

# reboot
```

## 创建和安装 guest

Perl 语言之父 Larry Wall 说过真正优秀的程序员有三大优良品质：偷懒，没有耐性和骄傲自大。所以能利用别人的劳动成果就不要自己重造轮子。我们可以到 [http://download.openvz.org/template/precreated/](http://download.openvz.org/template/precreated/) 下载已经安装好的模版，有 centos, debian, ubuntu, fedora, suse 等几个模版可以选择：

```
# cd /vz/template/cache
# wget http://download.openvz.org/template/precreated/ubuntu-10.04-x86.tar.gz
```

有了 ubuntu 10.04 的模版以后就可以用这个模版来创建 guest 系统（VE/VPS）了，以刚下载的 ubuntu-10.04-x86 为模版创建一个 ID 为 1 的 Virtual Environment (VE)，并指定 IP 地址、DNS 服务器地址、主机名、磁盘空间等，创建成功后启动 ID 为 1 的 VE，最后修改 root 密码：

```
# vzctl create 1 --ostemplate ubuntu-10.04-x86

# vzctl set 1 --onboot yes --save
# vzctl set 1 --ipadd 172.16.39.110 --save
# vzctl set 1 --nameserver 8.8.8.8 --save
# vzctl set 1 --hostname vps01.vpsee.com --save
# vzctl set 1 --diskspace 10G:10G --save
```

OK.到此虚拟机创建完成

下面启用虚拟机并设置虚拟机的密码

```
# vzctl start 1
# vzctl exec 1 passwd
```

启动、重启、关闭和断电关闭 ID 为 1 的 VE/VPS：

```
# vzctl start 1
# vzctl restart 1
# vzctl stop 1
# vzctl destroy 1
```

查看正在运行中的 VE/VPS：

```
# vzlist
      CTID      NPROC STATUS    IP_ADDR         HOSTNAME
         1          8 running   172.16.39.110   vps01.vpsee.com
```

计算 ID 为 1 的 VE/VPS 用到的资源：

```
# vzcalc -v 1
Resource     Current(%)  Promised(%)  Max(%)
Low Mem          0.06       1.44       1.44
Total RAM        0.19        n/a        n/a
Mem + Swap       0.08       1.30        n/a
Alloc. Mem       0.11       1.62       3.09
Num. Proc        0.01        n/a       0.32
--------------------------------------------
Memory           0.19       1.62       3.09
```

## 进入 guest

VE 成功启动后就可以进入系统了，相当于 xen 的 xm console，不过从 VE 退出来不需特殊按键直接 exit 就可以：

```
# vzctl enter 1
entered into CT 1
root@vps01:/# exit
logout
exited from CT 1
```

转自： [http://www.vpsee.com/2011/01/install-openvz-on-centos/](http://www.vpsee.com/2011/01/install-openvz-on-centos/)

================================================================

**安装OpenVZ的web管理界面**

上面的命令初次接触还是有一定难度的。为了方便使用直接装个web管理端。轻松集中管理大批量的OpevVZ服务器。

执行下面的命令：（漫长的等待吧）

[shell]wget -O – http://ovz-web-panel.googlecode.com/svn/installer/ai.sh | sh[/shell]

完成安装后浏览器打开：

[http://:3000][1]

[![openvz-web-panel](/wp-content/uploads/2013/11/openvz-web-panel.png)][2]

默认的用户名admin  admin登陆即可。

[![openvz-web-panel-1](http://blog.haohtml.com/wp-content/uploads/2013/11/openvz-web-panel-1.png)][3]

卸载：

[shell]wget -O – http://ovz-web-panel.googlecode.com/svn/installer/ai.sh | sh -s UNINSTALL=1[/shell]

转自： [http://blog.unixsir.net/?p=267](http://blog.unixsir.net/?p=267)

 [1]: http://%3Cyour-host%3E:3000%E2%80%B3>