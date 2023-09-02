---
title: CentOS升级内核及KVM安装
author: admin
type: post
date: 2010-10-26T00:51:18+00:00
url: /archives/6355
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - 内核

---
由于CentOS 默认内核为2.6.18，故需要升级内核

升级内核到2.6.27,很容易，但升级到2.6.28后的版本折腾了我很久

升级到2.6.27

> wget
> tar zxvf linux-2.6.27.tar.gz -C /usr/src
> cd /usr/src/linux-2.6.27
> make menuconfig
> make
> make modules_install
> cp arch/i386/boot/bzImage /boot/vmlinuz-2.6.27-root (注意:目录i386是根据你的系统类型, 如果是64位系统, 那就很可能是x86_64)
> cp System.map /boot/System.map-2.6.27-root
> mkinitrd /boot/initrd-2.6.27-root.img 2.6.27

vi /etc/grub.conf

> title CentOS (2.6.27)
> root (hd0,6)
> kernel /vmlinuz-2.6.27-root ro root=/dev/VolGroup00/LogVol00 rhgb quiet
> initrd /initrd-2.6.27-root.img
> title CentOS (2.6.18-92.1.18.el5)
> root (hd0,6)
> kernel /vmlinuz-2.6.18-92.1.18.el5 ro root=/dev/VolGroup00/LogVol00 rhgb quiet
> initrd /initrd-2.6.18-92.1.18.el5.img
> title CentOS (2.6.18-53.el5)
> root (hd0,6)
> kernel /vmlinuz-2.6.18-53.el5 ro root=/dev/VolGroup00/LogVol00 rhgb quiet
> initrd /initrd-2.6.18-53.el5.img
> title qrpeng windows xp
> rootnoverify (hd0,0)
> chainloader +1
>
> reboot

**升级到2.6.33**
由于需要安装LVS，需要内核版本为2.6.28以上，于是开始升级新内核，折腾了很久，最后按照如下步骤搞定的：

> wget
> tar zxvf linux-2.6.33.tar.gz -C /usr/src
> cd /usr/src/linux-2.6.33
> make distclean
> cp /boot/config-2.6.18-164.el5 .config
> make menuconfig
> 修改.config文件,在.config文件搜索CONFIG\_SYSFS\_DEPRECATED\_V2，会发现#CONFIG\_SYSFS\_DEPRECATED\_V2 is notset这一行，将该行修改为CONFIG\_SYSFS\_DEPRECATED_V2=y
> make all
> make modules_install
> make install

不要重新启动需要修改initrd

1.解压initrd

> cp /boot/initrd-2.6.33.img /tmp
> cd /tmp/
> mkdir newinitrd
> cd newinitrd/
> zcat ../initrd-2.6.33.img |cpio -i

2，编辑init，删掉重复的两行

> echo “Loading dm-region-hash.ko module”
> insmod /lib/dm-region-hash.ko
> echo “Loading dm-region-hash.ko module”
> insmod /lib/dm-region-hash.ko

3，重新打包initrd

> find .|cpio -c -o > ../initrd
> cd ..
> gzip -9 < initrd > initrd-2.6.33.img

initrd-2.6.33.img就是重新打包的initrd了，然后把initrd-2.6.33.img拷贝到/boot。

**安装kernel-devel**

到 或者相应版本即可

**安装KVM**

开启SELinux
（如果你的SELinux被禁用，virt-install将不会正常工作）

system-config-securitylevel-tui

kvm安装
(a)检查CPU是否支持硬件虚拟化-运行命令

egrep ‘(vmx|svm)’ –color=always /proc/cpuinfo

（如果输出的结果包含 vmx，它是 Intel；如果包含 svm，它是 AMD。如果你甚么都得不到，那应你的系统并没有支持虚拟化的处理。）

(b)安装KVM和virtinst（一个创建虚拟机的工 具），我们运行

yum install kvm kmod-kvm qemu libvirt python-virtinst

然后重新启动系统:

Reboot

**检查**
使用下列命令检查KVM是否成功安装

virsh -c qemu:///system list

将会显示如下结果:

[root@server1 ~]# virsh -c qemu:///system list

Id Name State

———————————-

[root@server1 ~]#

如果在这里显示的是一个错误的信息，说明有些东西出现了问题。

桥接网络
1)假如你想客端在内联网上以个别一台主机出现，让整个网络看得见，你需要采用桥接网络。

yum install bridge-utils2)创建文件/etc/sysconfig/network-scripts/ifcfg-br0

vi /etc/sysconfig/network-scripts/ifcfg-br0

DEVICE=br0

TYPE=Bridge

BOOTPROTO=static

BROADCAST=192.168.0.255

IPADDR=192.168.0.100

NETMASK=255.255.255.0

NETWORK=192.168.0.0

ONBOOT=yes

注意将上面的IP地址配置为你本机的IP地址

3）修改/etc/sysconfig/network-scripts/ifcfg-eth0，蓝色信息注释掉，红色为新添加

vi /etc/sysconfig/network-scripts/ifcfg-eth0

\# Realtek Semiconductor Co., Ltd. RTL-8139/8139C/8139C+

DEVICE=eth0

#BOOTPROTO=static

#BROADCAST=192.168.0.255

HWADDR=00:10:A7:05:AF:EB

#IPADDR=192.168.0.100

#NETMASK=255.255.255.0

#NETWORK=192.168.0.0

ONBOOT=yes

BRIDGE=br0

注意，此处一定要将eth0（即物理网卡，根据自己机器来确定名字）的IP地址要注释掉，否则配置不正确

KVM管理
GUI
安装 virt-viewer 或者virt-manager

yum install virt-viewer  virt-manager

以GUI安装OS
终端中输入 virt-manager 或者点击系统菜单：Applications—>System tools –>Virtual Machine Manager，将弹出GUI界面，利用该界面就很容易的安装新操作系统了。

注意，在此步执行前，请确保SELinux关闭或者是 permissive

管理客户机
可通过virt-manager或者virsh命令尽力管理

连接到virtual shell，运行

virsh –connect qemu:///system

进入后可用help命令寻求帮助

FAQ
如果新建客户机总是报错，说“Internal Error：Domain xxx didn’t show up”,是SELinux的问题，把SELinux设置permissive就可以了

**相关教程:**

[centos升级内核教程:http://blog.haohtml.com/archives/12448][1]

 [1]: http://blog.haohtml.com/archives/12448