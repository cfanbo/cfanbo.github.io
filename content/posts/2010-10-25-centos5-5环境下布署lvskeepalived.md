---
title: CentOS5.5环境下布署LVS+keepalived
author: admin
type: post
date: 2010-10-25T12:59:10+00:00
url: /archives/6348
IM_data:
 - 'a:11:{s:53:"http://img1.51cto.com/attachment/201010/154055336.jpg";s:69:"http://blog.haohtml.com/wp-content/uploads/2011/07/8e5f_154055336.jpg";s:53:"http://img1.51cto.com/attachment/201010/154138720.jpg";s:69:"http://blog.haohtml.com/wp-content/uploads/2011/07/4781_154138720.jpg";s:53:"http://img1.51cto.com/attachment/201010/154209809.jpg";s:69:"http://blog.haohtml.com/wp-content/uploads/2011/07/9671_154209809.jpg";s:53:"http://img1.51cto.com/attachment/201010/154241173.jpg";s:69:"http://blog.haohtml.com/wp-content/uploads/2011/07/c97a_154241173.jpg";s:53:"http://img1.51cto.com/attachment/201010/154407568.jpg";s:69:"http://blog.haohtml.com/wp-content/uploads/2011/07/80f3_154407568.jpg";s:53:"http://img1.51cto.com/attachment/201010/154438783.jpg";s:69:"http://blog.haohtml.com/wp-content/uploads/2011/07/9c9a_154438783.jpg";s:53:"http://img1.51cto.com/attachment/201010/154506634.jpg";s:69:"http://blog.haohtml.com/wp-content/uploads/2011/07/4fda_154506634.jpg";s:53:"http://img1.51cto.com/attachment/201010/154530799.jpg";s:69:"http://blog.haohtml.com/wp-content/uploads/2011/07/ca66_154530799.jpg";s:53:"http://img1.51cto.com/attachment/201010/154552509.jpg";s:69:"http://blog.haohtml.com/wp-content/uploads/2011/07/0de4_154552509.jpg";s:53:"http://img1.51cto.com/attachment/201010/154613452.jpg";s:69:"http://blog.haohtml.com/wp-content/uploads/2011/07/7527_154613452.jpg";s:53:"http://img1.51cto.com/attachment/201010/154632533.jpg";s:69:"http://blog.haohtml.com/wp-content/uploads/2011/07/ebff_154632533.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - centos
 - keepalived
 - lvs

---
#!/bin/bash
\# BY kerryhu
\# MAIL:king_819@163.com
\# BLOG:http://kerry.blog.51cto.com
\# Please manual operation yum of before Operation…..
系统环境\`：CentOS 5.5（定制安装）
**组件：**
Base
Development Libraries
Development Tools
Editors
Text-based Internet

lvs-master：192.168.9.201
lvs-backup：192.168.9.202
vip：192.168.9.200
web1：192.168.9.203
web2：192.168.9.204
netmask：255.255.255.0
gateway：192.168.9.1

**网络拓扑：**

[![](http://img1.51cto.com/attachment/201010/154055336.jpg)](http://img1.51cto.com/attachment/201010/154055336.jpg)

echo “============================ 更新系统时间 ======================”
yum install -y ntp
ntpdate time.nist.gov
echo “00 01 \* \* * /usr/sbin/ntpdate time.nist.gov” /etc/crontab

echo “============================ 关闭不用服务 =======================”
/root/del_servcie.sh           # 附件中自定义脚本

**一.下面开始LVS的配置:**

echo “========================= 安装ipvsadm、keepalived ==================”
[root@master ~]# cd /usr/local/src
[root@master ~]# wget
[root@master ~]# ln -sv /usr/src/kernels/2.6.18-194.el5-i686/ /usr/src/linux
[root@master ~]# tar -zxvf ipvsadm-1.24.tar.gz
[root@master ~]# cd ipvsadm-1.24
[root@master ~]# make;make install
[root@master ~]# cd ..

[root@master ~]# wget
[root@master ~]# tar -zxvf keepalived-1.1.17.tar.gz
[root@master ~]# cd keepalived-1.1.17
[root@master ~]# ./configure
configure: error:
!!! OpenSSL is not properly installed on your system. !!!
!!! Can not include OpenSSL headers files.
解决办法：
[root@master ~]# yum -y install openssl-devel
[root@master ~]# ./configure
[root@master ~]# make;make install
编译的时候出现这个提示，说明keepalived和内核结合了，如果不是这样的，需要加上这个参数./configure –with-kernel-dir=/kernel/path

Keepalived configuration

————————
Keepalived version       : 1.1.17
Compiler                 : gcc
Compiler flags           : -g -O2
Extra Lib                : -lpopt -lssl -lcrypto
Use IPVS Framework       : Yes
IPVS sync daemon support : Yes
Use VRRP Framework       : Yes
Use LinkWatch            : No
Use Debug flags          : No

echo “======================= 配置keepalived ===========================”
[root@**master ~**]#  cp /usr/local/etc/rc.d/init.d/keepalived /etc/rc.d/init.d/
[root@master ~]#  cp /usr/local/etc/sysconfig/keepalived /etc/sysconfig/
[root@master ~]#  mkdir /etc/keepalived
[root@master ~]#  cp /usr/local/sbin/keepalived /usr/sbin/
[root@master ~]# vi /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
notification_email {

}
notification\_email\_from
smtp_server smtp.163.com
\# smtp\_connect\_timeout 30
router\_id LVS\_DEVEL
}

\# VIP1
vrrp\_instance VI\_1 {
state MASTER             #备份服务器上将MASTER改为BACKUP
interface eth0
lvs\_sync\_daemon_inteface eth0
virtual\_router\_id 51
priority 100    # 备份服务上将100改为90
advert_int 5
authentication {
auth_type PASS
auth_pass 1111
}
virtual_ipaddress {
192.168.9.200
#(如果有多个VIP，继续换行填写.)
}
}

virtual_server 192.168.9.20080 {
delay_loop 6                  #(每隔10秒查询realserver状态)
lb_algo wlc                  #(lvs 算法)
lb_kind DR                  #(Direct Route)
persistence_timeout 60        #(同一IP的连接60秒内被分配到同一台realserver)
protocol TCP                #(用TCP协议检查realserver状态)

real_server 192.168.9.203 80 {
weight 100               #(权重)
TCP_CHECK {
connect_timeout 10       #(10秒无响应超时)
nb\_get\_retry 3
delay\_before\_retry 3
connect_port 80
}
}
real_server 192.168.9.204 80 {
weight 100
TCP_CHECK {
connect_timeout 10
nb\_get\_retry 3
delay\_before\_retry 3
connect_port 80
}
}
}
[root@master ~]#  service keepalived start|stop
[root@master ~]# chkconfig –level 2345 keepalived on

**二.配置后端的realserver,这里为web\_1和web\_2**

echo “====================== 配置realserver =========================”
[root@**web_1** ~]# vi /root/lvs_real.sh
#!/bin/bash
\# description: Config realserver
#Written by : [http://kerry.blog.51cto.com][1]

SNS_VIP=192.168.9.200

/etc/rc.d/init.d/functions

case “$1” in
start)
/sbin/ifconfig lo:0 $SNS\_VIP netmask 255.255.255.255 broadcast $SNS\_VIP
/sbin/route add -host $SNS_VIP dev lo:0
echo “1” >/proc/sys/net/ipv4/conf/lo/arp_ignore
echo “2” >/proc/sys/net/ipv4/conf/lo/arp_announce
echo “1” >/proc/sys/net/ipv4/conf/all/arp_ignore
echo “2” >/proc/sys/net/ipv4/conf/all/arp_announce
sysctl -p >/dev/null 2>&1
echo “RealServer Start OK”

;;
stop)
/sbin/ifconfig lo:0 down
/sbin/route del $SNS_VIP >/dev/null 2>&1
echo “0” >/proc/sys/net/ipv4/conf/lo/arp_ignore
echo “0” >/proc/sys/net/ipv4/conf/lo/arp_announce
echo “0” >/proc/sys/net/ipv4/conf/all/arp_ignore
echo “0” >/proc/sys/net/ipv4/conf/all/arp_announce
echo “RealServer Stoped”
;;
*)
echo “Usage: $0 {start|stop}”
exit 1
esac

exit 0

[root@**web_1 ~**]# chmod +x /roo/lvs_real.sh
[root@web\_1 ~]# /root/lvs\_real.sh start
[root@web_1 ~]# ifconfig [![](http://img1.51cto.com/attachment/201010/154138720.jpg)](http://img1.51cto.com/attachment/201010/154138720.jpg)

[root@web\_1 ~]# echo “/root/lvs\_real.sh start” >> /etc/rc.local

**三.测试LVS**

echo “===================== 测试LVS+keepalived ========================”

**1.首先测试lvs主备的高可用性(HA)**

#LVS\_master、LVS\_backup上开启keepalived，LVS_master先绑定VIP
LVS_master： [![](http://img1.51cto.com/attachment/201010/154209809.jpg)](http://img1.51cto.com/attachment/201010/154209809.jpg) LVS_backup： [![](http://img1.51cto.com/attachment/201010/154241173.jpg)](http://img1.51cto.com/attachment/201010/154241173.jpg)

#解析域名，测试访问，LVS转发

[![](http://img1.51cto.com/attachment/201010/154407568.jpg)](http://img1.51cto.com/attachment/201010/154407568.jpg)

#测试关闭LVS\_master，短暂的掉包后，LVS\_backup马上接替工作

[![](http://img1.51cto.com/attachment/201010/154438783.jpg)](http://img1.51cto.com/attachment/201010/154438783.jpg)
LVS\_backup接替LVS\_master绑定VIP

以下为vip_bacup机器的ip信息: [![](http://img1.51cto.com/attachment/201010/154506634.jpg)](http://img1.51cto.com/attachment/201010/154506634.jpg)

LVS_backup负责转发

[![](http://img1.51cto.com/attachment/201010/154530799.jpg)](http://img1.51cto.com/attachment/201010/154530799.jpg)

LVS_master重启完成后，就会自动接回控制权，继续负责转发

[![](http://img1.51cto.com/attachment/201010/154552509.jpg)](http://img1.51cto.com/attachment/201010/154552509.jpg) **2.测试realserver的高可用性(HA)**

#测试关闭其中一台realserver

[![](http://img1.51cto.com/attachment/201010/154613452.jpg)](http://img1.51cto.com/attachment/201010/154613452.jpg)

通过上面测试可以知道，当realserver故障或者无法提供服务时，负载均衡器通过健康检查自动把失效的机器从转发队列删除掉，实现故障隔离，保证用户的访问不受影响.

#重启被关闭的realserver

[![](http://img1.51cto.com/attachment/201010/154632533.jpg)](http://img1.51cto.com/attachment/201010/154632533.jpg)

当realserver故障恢复后，负载均衡器通过健康检查自动把恢复后的机器添加到转发队列中

本文出自 “[聆听未来][1]” 博客，请务必保留此出处

**相关教程:**

linux下利用Haproxy和keepalived实现简单负载均衡: [http://blog.haohtml.com/archives/10975](http://blog.haohtml.com/archives/10975)

 [1]: http://kerry.blog.51cto.com/