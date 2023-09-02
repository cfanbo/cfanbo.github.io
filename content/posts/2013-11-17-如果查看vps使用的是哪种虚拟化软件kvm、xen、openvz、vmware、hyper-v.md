---
title: 如果查看vps使用的是哪种虚拟化软件(kvm、xen、openvz、vmware、Hyper-V、HVM)
author: admin
type: post
date: 2013-11-17T09:02:29+00:00
url: /archives/14756
categories:
 - 服务器
tags:
 - 虚拟化
 - vps

---
现在的VPS市场鱼龙混杂，如何检测自己购买的VPS是否如服务商列举出来的参数和配置以及环境呢？
如果你要检测自己购买的是否为真的Xen，可以用如下方法进行测试，比较专业的就是用virt-what脚本进行检测：

[shell]wget http://people.redhat.com/~rjones/virt-what/files/virt-what-1.12.tar.gz
tar zxvf virt-what-1.12.tar.gz
cd virt-what-1.12/
./configure
make && make install
virt-what[/shell]

如果是Xen的VPS，则会返回如下信息：

xen
xen-domU

如果是vmware的话，会返回 vmware.

对于想知道自己用的服务器是不是vps,也可以用这种办法来判断的．