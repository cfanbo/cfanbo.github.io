---
title: Nginx+keepalived负载均衡篇
author: admin
type: post
date: 2010-08-28T13:22:39+00:00
url: /archives/5367
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - keepalived
 - nginx

---
由于nginx的url hash功能可以很好的提升squid的性能，所以我把squid前端的负载均衡器更换为nginx，但是一台nginx就形成了单点，现在使用keepalived来解决这个问题，keepalived的故障转移时间很短，而且配置简单，这也是选择keepalived的一个主要原因，建议日PV值小的中小型企业web均可采用如下方案实行，下面直接上安装步骤：

**一、****环境****：**

> centos5.3、nginx-0.7.51、keepalived-1.1.19
>
> 主nginx负载均衡器：192.168.0.154
>
> 辅nginx负载均衡器：192.168.9.155
>
> vip：192.168.0.188

**二、安装keepalived**

> #tar zxvf keepalived-1.1.19.tar.gz
>
>
> #cd keepalived-1.1.19
>
>
> #./configure –prefix=/usr/local/keepalived
>
>
> #make
>
>
> #make install
>
>
> #cp /usr/local/keepalived/sbin/keepalived /usr/sbin/
>
>
> #cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/
>
>
> #cp /usr/local/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/
>
>
> #mkdir /etc/keepalived
>
>
> #cd /etc/keepalived/

**vim keepalived.conf**

> ! Configuration File for keepalived
>
>
> global_defs {
>
> notification_email {
>
>
> yuhongchun027@163.com
>
>
> }
>
>
> notification_email_from keepalived@chtopnet.com
>
>
> smtp_server 127.0.0.1
>
>
> smtp_connect_timeout 30
>
>
> router_id LVS_DEVEL
>
>
> }
>
>
> vrrp_instance VI_1 {
>
>
> state MASTER
>
>
> interface eth0
>
>
> virtual_router_id 51
>
>
> mcast_src_ip 192.168.0.154    <==主nginx的IP地址
>
>
> priority 100
>
>
> advert_int 1
>
>
> authentication {
>
>
> auth_type PASS
>
>
> auth_pass chtopnet
>
>
> }
>
>
> virtual_ipaddress {
>
>
> 192.168.0.188                      <==VIP地址
>
>
> }
>
>
> }

**#service keepalived start**

我们来看一下日志：

[root@ltos ~]# tail /var/log/messages

> Oct 6 03:25:03 ltos avahi-daemon[2306]: Registering new address record for 192.168.0.188 on eth0.
>
>
> Oct 6 03:25:03 ltos avahi-daemon[2306]: Registering new address record for 192.168.0.154 on eth0.
>
>
> Oct 6 03:25:03 ltos avahi-daemon[2306]: Registering HINFO record with values ‘I686’/’LINUX’.
>
>
> Oct 6 03:25:23 ltos avahi-daemon[2306]: Withdrawing address record for fe80::20c:29ff:feb9:eeab on eth0.
>
>
> Oct 6 03:25:23 ltos avahi-daemon[2306]: Withdrawing address record for 192.168.0.154 on eth0.
>
>
> Oct 6 03:25:23 ltos avahi-daemon[2306]: Host name conflict, retrying with
>
> Oct 6 03:25:23 ltos avahi-daemon[2306]: Registering new address record for fe80::20c:29ff:feb9:eeab on eth0.
>
>
> Oct 6 03:25:23 ltos avahi-daemon[2306]: Registering new address record for 192.168.0.188 on eth0.
>
>
> Oct 6 03:25:23 ltos avahi-daemon[2306]: Registering new address record for 192.168.0.154 on eth0.
>
>
> Oct 6 03:25:23 ltos avahi-daemon[2306]: Registering HINFO record with values ‘I686’/’LINUX’.

很显然vip已经启动，我们还可以通过命令：**#ip a** 来检查

[root@ltos html]# ip a

> 1: lo:  mtu 16436 qdisc noqueue
>
>
> link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>
>
> inet 127.0.0.1/8 scope host lo
>
>
> inet6 ::1/128 scope host
>
>
> valid_lft forever preferred_lft forever
>
>
> 2: eth0:  mtu 1500 qdisc pfifo_fast qlen 1000
>
>
> link/ether 00:0c:29:ba:9b:e7 brd ff:ff:ff:ff:ff:ff
>
>
> inet 192.168.0.154/24 brd 192.168.0.255 scope global eth0
>
>
> inet 192.168.0.188/32 scope global eth0
>
>
> inet6 fe80::20c:29ff:feba:9be7/64 scope link
>
>
> valid_lft forever preferred_lft forever
>
>
> 3: sit0:  mtu 1480 qdisc noop
>
>
> link/sit 0.0.0.0 brd 0.0.0.0

说明vip已经启动，这样主服务器就配置好了，辅机的配置大致一样，除了配置文件有少部分的变化，下面贴出辅机的配置文件：

> ! Configuration File for keepalived
>
>
> global_defs {
>
> notification_email {
>
>
> yuhongchun027@163.com
>
>
> }
>
>
> notification_email_from keepalived@chtopnet.com
>
>
> smtp_server 127.0.0.1
>
>
> smtp_connect_timeout 30
>
>
> router_id LVS_DEVEL
>
>
> }
>
>
> vrrp_instance VI_1 {
>
>
> state BACKUP
>
>
> interface eth0
>
>
> virtual_router_id 51
>
>
> mcast_src_ip 192.168.0.155             <==从nginx的IP的地址
>
>
> priority 100
>
>
> advert_int 1
>
>
> authentication {
>
>
> auth_type PASS
>
>
> auth_pass chtopnet
>
>
> }
>
>
> virtual_ipaddress {
>
>
> 192.168.0.188
>
>
> }
>
>
> }

检查其配置

[root@ltos html]# ip a

> 1: lo:  mtu 16436 qdisc noqueue
>
>
> link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>
>
> inet 127.0.0.1/8 scope host lo
>
>
> inet6 ::1/128 scope host
>
>
> valid_lft forever preferred_lft forever
>
>
> 2: eth0:  mtu 1500 qdisc pfifo_fast qlen 1000
>
>
> link/ether 00:0c:29:ba:9b:e7 brd ff:ff:ff:ff:ff:ff
>
>
> inet 192.168.0.155/24 brd 192.168.0.255 scope global eth0
>
>
> inet 192.168.0.188/32 scope global eth0
>
>
> inet6 fe80::20c:29ff:feba:9be7/64 scope link
>
>
> valid_lft forever preferred_lft forever
>
>
> 3: sit0:  mtu 1480 qdisc noop
>
>
> link/sit 0.0.0.0 brd 0.0.0.0

测试其效果方法很简单，分别在主辅机上建立不同的主页，index.html的内容分别为192.168.0.154,192.168.0.155，然后用客户机上elinks [http://192.168.0.188](http://192.168.0.188/),主机down掉后辅机会马上接替提供服务，间隔时间几乎无法感觉出来,如有疑问请联系 [yuhongchun027@163.com](mailto:yuhongchun027@163.com)(抚琴煮酒)