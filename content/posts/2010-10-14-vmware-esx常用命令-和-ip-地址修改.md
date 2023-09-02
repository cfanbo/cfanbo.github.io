---
title: VMware ESX常用命令 和 IP 地址修改
author: admin
type: post
date: 2010-10-14T15:12:28+00:00
url: /archives/6090
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ESX
 - vmware

---
一． VMware ESX Command

1. 看你的esx版本

vmware –v

2. 查看显示ESX硬件，内核，存储，网络等信息

esxcfg-info -a（显示所有相关的信息）

esxcfg-info -w（显示esx上硬件信息）

3. 列出esx里知道的服务

esxcfg-firewall –s

4. 查看具体服务的情况

esxcfg-firewall -q sshclinet

5. 重新启动vmware服务

service mgmt-vmware restart

6. 修改root的密码

passwd root

7. 设置kernel高级选项

esxcfg-advcfg -d（将系统内核恢复默认值）

8. 管理资源组

esxcfg-resgrp -l（显示所有资源组）

9. 列出你当前的虚拟交换机

esxcfg-vswitch -l

esxcfg-vswitch -v 10 -p “Service Console” vSwitch0 (将vSwitch0上的Service Console划分到vLan 10上，如果vLan号为0则不设置vLan)

10. 查看控制台的设置

esxcfg-vswif -l  （列出已添加的网卡）

esxcfg-vswif -a （添加网卡）

11. 列出系统的网卡

esxcfg-nics –l

12. 添加一个虚拟交换机，名字叫（internal）连接到两块物理网卡，（重新启动服务，vi就能看见了）

esxcfg-vswitch -a vSwitch1

esxcfg-vswitch -A internal vSwitch1

esxcfg-vswitch -L vmnic1 vSwitch1

esxcfg-vswitch -L vmnic2 vSwitch1

13.  删除交换机,(注意，别把控制台的交换机也删了）

esxcfg-vswitch -D vSwitch1

14.  删除交换机上的网卡

esxcfg-vswitch -u vmnic1 vswitch2

15.  删除portgroup

esxcfg-vswitch -D internel vswitch1

16.  创建 vmkernel switch，如果你希望使用vmotion，iscsi的这些功能，你必须创建(通常是不需要添加网关的）

esxcfg-vswitch -l

esxcfg-vswitch -a vswitch2

esxcfg-vswitch -A “vm kernel” vswitch2

esxcfg-vswitch -L vmnic3 vswitch2

esxcfg-vmknic -a “vm kernel” -i 172.16.1.141 -n 255.255.252.0 （添加一个vmkernel）

17.  防火墙设置

esxcfg-firewall -e sshclient （打开防火墙ssh端口）

esxcfg-firewall -d sshclient （关闭防火墙ssh端口）

esxcfg-firewall -e veritasNetBackup（允许Veritas Netbackup服务）

esxcfg-firewall -o 123，udp，out，ntp（为ntp服务打开UDP协议中的123端口的输出）

18.  路由管理

esxcfg-route（VM生成网卡的路由管理）

esxcfg-route（显示路由表）

esxcfg-route 172.16.0.254（设置vmkernel网关）

19.  创建控制台

esxcfg-vswitch -a vSwitch0

esxcfg-vswitch -A “service console” vSwitch0

esxcfg-vswitch -L vmnic0 vSwitch0

esxcfg-vswif -a vswif0 -p “service console” -i 172.16.1.140 -n 255.255.252.0

20.  添加nas设备(a添加标签，-o，是nas服务器的名字或ip，-s是nas输入的共享名字）

esxcfg-nas -a isos -o nas.vmwar.cn -s isos

21.  nas连接管理

esxcfg-nas -r （强迫esx去连接nas服务器）

esxcfg-nas -l   (用esxcfg-nas -l来看看结果）

esxcfg-nas -a（添加NAS文件系统到/vmfs目录下）

esxcfg-nas -d（删除NAS文件系统）

22.  扫描SCSI设备上的LUN信息

esxcfg-rescan

23.  连接iscsi设备(e:enable q:查询 d, disable s:强迫搜索）

esxcfg-swiscsi -e

24.  设置targetip

vmkiscsi-tool -D -a 172.16.1.133 vmhba40

25.  列出和target的连接

vmkiscsi-tool -l -T vmhba40

26.  列出当前的磁盘

ls -l /vmfs/devices/disks

27.  内核dump管理工具

esxcfg-dumppart -l（显示当前dump分区配置信息）

28.  路径管理

esxcfg-mpath -l（显示所有路径）

esxcfg-mpath -a（显示所有HBA卡）

29.  ESX授权管理配置

esxcfg-auth

esxcfg-auth –enablenis（运行NIS验证）

30.  管理启动设备

esxcfg-boot

esxcfg-boot -b（更新启动设备）

31.  执行initrd的初始化设置

esxcfg-init

esxcfg-init（初始化设备）

32.  esxcfg-linuxnet（在linux debug模式中，转换vswif设备命名为linux自带的eth命名规则）

esxcfg-linuxnet –setup

33.  升级

esxcfg-upgrade（ESX2.X升级到ESX3.X）

二． 使用命令更改Service Console IP

在CLI下更改service console的ip地址，注意大小写，vmware是把物理nic虚拟成vmnic，在vmnic上创建虚拟交换机vswitch，是把网卡当成交换机来使用，不能对网卡进行ip地址的设置，只能在vswitch上创建interface就是vswif，对vswif进行ip设置

1.  使用CLI创建Service Console

[root@VI3 root]# esxcfg-vswitch -a vSwitch0                     #创建vSwitch0

[root@VI3 root]# esxcfg-vswitch -A “Service Console” vSwitch0   #在vSwitch0上创建Portgroup,命名为Service Console

[root@VI3 root]# esxcfg-vswitch -L vmnic0 vSwitch0              #将vmnic0绑定在vSwitch0

[root@VI3 root]# esxcfg-vswitch –l         #可以看到service console已经绑定 vmnic0

Switch Name   Num Ports  Used Ports Configured Ports MTU    Uplinks

vSwitch0      64         5          64               1500   vmnic0

PortGroup Name   VLAN ID    Used Ports Uplinks

Service Console    0         1          vmnic0

[root@VI3 root]# esxcfg-vswif -a vswif0 -p “Service Console” -i 192.168.1.1 -n 255.255.255.0

#创建vswif0并与service console绑定,在ESX里ip地址只能跟vswif0绑定,也就是虚拟交换机的interface

[root@VI3 root]# esxcfg-vswif –l     #可以看到Service console的IP已经配置到vswif0

Name    Port Group      IP Address    Netmask         Broadcast      Enabled  DHCP

vswif0  Service Console 192.168.1.50   255.255.255.0    192.168.1.255  true  false

[root@VI3 root]# esxcfg-vswitch –l

Switch Name   Num Ports  Used Ports Configured Ports MTU    Uplinks

vSwitch0      64         5          64               1500   vmnic0

PortGroup Name   VLAN ID    Used Ports Uplinks

Service Console    0         1          vmnic0

[root@VI3 root]# service mgmt-vmware restart          #重启服务,到这里正常情况下就可以使用VI连接到ESX

————–↓如果不小心配置错了要删除，请看下面↓—————

[root@VI3 root]# esxcfg-vswif –l   #vswif0代表的虚拟网卡的interface0，service console对应vswif0

Name   Port Group      IP Address    Netmask       Broadcast     Enabled  DHCP

vswif0 Service Console 192.168.1.1  255.255.255.0  192.168.1.255  true    false

[root@VI3 root]# esxcfg-vswif -d vswif0                 #删除vswif0

[root@VI3 root]# esxcfg-vswitch -l

Switch Name   Num Ports  Used Ports Configured Ports MTU    Uplinks

vSwitch0      64         5          64               1500   vmnic0

PortGroup Name   VLAN ID    Used Ports Uplinks

Service Console    0         1          vmnic0

[root@VI3 root]# esxcfg-vswitch –D “Service Console” vSwitch0    #删除vSwitch0上面portgroup

[root@VI3 root]# esxcfg-vswitch –D “VM Network” vSwitch0

[root@VI3 root]# esxcfg-vswitch -d vSwitch0                #删除vswitch0

[root@VI3 root]# esxcfg-vswitch –l           #之前操作删除了vswitch信息,现在是空白

Switch Name   Num Ports  Used Ports Configured Ports MTU    Uplinks

PortGroup Name   VLAN ID    Used Ports Uplinks

2.  如果不行检查一下以下配置文件.

[root@VI3 root]# vi /etc/sysconfig/network                 #这里纪录主机名字和网关

NETWORKING=yes

HOSTNAME=VI3

GATEWAY=192.168.251.12         #网关

GATEWAYDEV=vswif0                #网关指定在vswif0

[root@VI3 root]# vi /etc/sysconfig/network-scripts/ifcfg-vswif0        #看看这里的信息是否跟之前配置吻合

DEVICE=vswif0                        #之前把service cosole与vswif0关联

MACADDR=00:50:56:43:a3:52

PORTGROUP=portgroup6     #这里的protgroup与service console一致

BOOTPROTO=static

BROADCAST=192.168.251.255

IPADDR=192.168.251.60                        #与service console一致

NETMASK=255.255.255.0

ONBOOT=yes

如果以上不一致,可以手动更改

在vi编辑器中,i键是插入模式,进行文本更改,esc键退出插入模式,:wq保存并退出.

编辑完成reboot.可能启动后显示地址跟设置不同,但是可以使用VI连接到ESX

补：如果只想修改Service Console的IP可以直接执行以下命令：

esxcfg-vswif -i xxx.xxx.xxx.xxx vswif