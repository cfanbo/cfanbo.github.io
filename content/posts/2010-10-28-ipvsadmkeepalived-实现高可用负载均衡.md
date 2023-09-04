---
title: ipvsadm+keepalived 实现高可用负载均衡
author: admin
type: post
date: 2010-10-28T03:27:37+00:00
url: /archives/6404
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - 负载均衡
 - ipvsadm
 - keepalived
 - lvs

---
#### 一.使用系统Red Hat Enterprise Linux Server release 5.4

#### 二.安装环境

#### 1.说明

realserver:192.168.1.11

realserver:192.168.1.12

lvs控制机 MASTER:192.168.1.100

BACKUP:192.168.1.101

虚拟VIP：192.168.1.200

其中：realserver上只需要简单的安装apache即可

lvs控制机需要安装：ipvsadm，keepalived

#### 2.lvs控制机安装，主备机分别安装ipvsadm

实现LVS/DR最重要的两个东西是ipvs内核模块和ipvsadm工具包，现在的系统已经包含ip_vs模块

#### 1）检查内核模块，看一下ip_vs 是否被加载

> # lsmod |grep ip_vs

>ip_vs    35009    0

如果没有显示，则说明没有加载，执行命令 modprobe ip_vs 就可以把ip_vs模块加载到内核

> #modprobe ip_vs

#### 2）安装ipvsadm

先把目录/usr/src/kernels/2.6.18-164.el5-x86_64链接为/usr/src/linux，命令如下

> ln –s /usr/src/kernels/2.6.18-164.el5-x86_64 /usr/src/linux

解压ipvsadm-1.24.tar.gz,执行”make;make install”完成安装。

#### 3）lvs/dr脚本，在主备机上备份部署

vi lvsdr

> #### #!/bin/bash

 RIP1=192.168.1.11

 RIP2=192.168.1.12
>
> VIP1=192.168.1.200
>
> ####

/etc/rc.d/init.d/functions
>
> case “$1″ in
>
> start)
> echo ” start LVS of DirectorServer”
>
> \# set the Virtual IP Address and sysctl parameter
> /sbin/ifconfig eth0:0 $VIP1 broadcast $VIP1 netmask 255.255.255.255 up
> #/sbin/ifconfig eth0:1 $VIP2 broadcast $VIP2 netmask 255.255.255.255 up
> /sbin/route add -host $VIP1 dev eth0:0
> #/sbin/route add -host $VIP2 dev eth0:1
> echo “1” >/proc/sys/net/ipv4/ip_forward
>
> #Clear IPVS table
> /sbin/ipvsadm -C
>
> #set LVS
> #Web Apache
> /sbin/ipvsadm -A -t $VIP1:80 -s rr -p 120
> /sbin/ipvsadm -a -t $VIP1:80 -r $RIP1:80 -g
> /sbin/ipvsadm -a -t $VIP1:80 -r $RIP2:80 -g
>
> #Run LVS
> /sbin/ipvsadm
> ;;
> stop)
> echo “close LVS Directorserver”
> echo “0” >/proc/sys/net/ipv4/ip_forward
> /sbin/ipvsadm -C
> /sbin/ifconfig eth0:0 down
> ;;
> *)
> echo “Usage: $0 {start|stop}”
> exit 1
> esac

#### 4)主备机上分别安装keepalived

> #### tar zxf keepalived-1.1.17.tar.gz
>
> #### cd keepalived-1.1.17
>
> #### ./configure –prefix=/usr/local/keepalive -with-kernel-dir=/usr/src/kernels/path
>
> #### make && make install
>
> #### mkdir -p /etc/keepalived/

#### vi keepalived.conf以下配置文件

> #### ! Configuration File for keepalived

 global_defs {

 router_id LVS_DEVEL

 }
>
> vrrp\_instance VI\_1 {
> state MASTER         #备份服务器上将MASTER改为BACKUP
> interface eth0       #HA监测网络接口
> virtual\_router\_id 51 #主、备机的virtual\_router\_id必须相同
> priority 90          #主、备机取不同的优先级，主机值较大，备份机值较小
> advert_int 1         #VRRP Multicast广播周期秒数
> authentication {
> auth_type PASS   #VRRP认证方式
> auth_pass 1111   #VRRP口令字
> }
> virtual_ipaddress {
> 192.168.1.200    #LVS虚拟地址
> }
> }
>
> virtual_server 192.168.1.200 80 {
> delay_loop 2
> lb_algo rr
> lb_kind DR
> protocol TCP
>
> real_server 192.168.1.11 80 {
> weight 1
> TCP_CHECK {
> connect_timeout 3
> nb\_get\_retry 3
> delay\_before\_retry 3
> }
> }
> real_server 192.168.1.12 80 {
> weight 1
> TCP_CHECK {
> connect_timeout 3
> nb\_get\_retry 3
> delay\_before\_retry 3
> }
> }
> }

#### 三.分别启动主备机上的lvs脚本和keepalived

> #### sh lvsdr start
>
> #### /usr/local/keepalived/sbin/keepalived -D

#### 四.测试

#### 停掉主机ipvsadmn和keepalived，从机立即接管服务。

#### 启动主机服务，主机负载均衡生效。

这里采用了“rr:轮叫(Round Robin)” 算法，容易看出实际效果，对它的解释如下：

调度器通过”轮叫”调度算法将外部请求按顺序轮流分配到集群中的真实服务器上，它均等地对待每一台服务器，而不管服务器上实际的连接数和系统负载.

#### 如需了解其它ipvsadm的lvs多种算法，请参考： [http://blog.haohtml.com/archives/6381](http://blog.haohtml.com/archives/6381)