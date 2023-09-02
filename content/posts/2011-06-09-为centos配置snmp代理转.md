---
title: 为CentOS配置snmp代理(转)
author: admin
type: post
date: 2011-06-09T06:32:22+00:00
url: /archives/9735
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - snmp

---
切换到系统管理员帐户

**安装snmp**
确认snmp代理已安装

> rpm -q net-snmp

如果未安装，安装snmp

> yum install net-snmp

设置开机自动运行snmp

> /sbin/chkconfig snmpd on

**配置snmp**
编辑/etc/snmp/snmpd.conf

更改团体名
查找如下行
\# sec.name source community
com2sec notConfigUser default public
将团体名public改为其它任意字段，例：
com2sec notConfigUser default monit

给予可读权限
查找如下行
\# group context sec.model sec.level prefix read write notif
access notConfigGroup “” any noauth exact systemview none none
将read权限systemview改为all，例：
access notConfigGroup “” any noauth exact all none none
查找如下行
\## incl/excl subtree mask
#view all included .1 80
去掉#view all前面的#，例：
view all included .1 80

**启动snmp**

> ****/etc/init.d/snmpd start

如果已启动则重启snmp服务

> /etc/init.d/snmpd restart

**测试snmp**
查看端口是否打开

> netstat -ln | grep 161

安装snmp测试工具

> yum install net-snmp-utils

本机测试snmp数据（修改monit为配置的团体名）

> snmpwalk -v 2c -c monit localhost system

远程测试snmp数据（修改ip为服务器ip，snmpwalk命令需要安装net-snmp）

> snmpwalk -v 2c -c monit ip system

**错误排除**
防火墙禁止访问
如果本地测试snmp有数据，远程测试snmp无数据则由于服务器防火墙禁止了外部访问服务器udp 161端口，则：
修改 /etc/sysconfig/iptables （或者：/etc/sysconfig/iptables-config ） ，增加如下规则：

> -A RH-Firewall-1-INPUT -p udp -m state –state NEW -m udp –dport 161 -j ACCEPT

重启iptables

> /etc/init.d/iptables restart



**snmpwalk参数的使用详解：**

```
snmpwalk -v 2c -c public 10.103.33.1 .1.3.6.1.2.1.25.1    得到取得windows端的系统进程用户数等

其中-v是指版本,-c 是指密钥,也就是客户端snmp.conf里面所设置的,下面类同.

2、snmpwalk -v 2c -c public 10.103.33.1 .1.3.6.1.2.1.25.2.2  取得系统总内存

3、snmpwalk -v 2c -c public 10.103.33.1 hrSystemNumUsers  取得系统用户数

4、snmpwalk -v 2c -c public 10.103.33.1 .1.3.6.1.2.1.4.20    取得IP信息

5、snmpwalk -v 2c -c public 10.103.33.1 system   查看系统信息

6、snmpwalk -v 2c -c public 10.103.33.1 ifDescr 获取网卡信息

以上只是一些常用的信息,snmpwalk功能很多,可以获取系统各种信息,只要更改后面的信息类型即可.如果不知道什么类型,也可以不指定,这样所有系统信息都获取到:

snmpwalk -v 2c -c public 10.103.33.1
```

SNMP协议报文字段中的团体名是什么意思

假设我要知道你现在穿的衣服的大小需要通过密码访问 那么会有两种密码，一种是Read可读，一种是Write可读可写 团体名就是这样两种密码，默认是Read=Public,Write=Private 只有持有read这个密码，才能知道你衣服的尺寸。 只有持有Write这个密码，才能修改你的衣服尺寸。 比喻不太恰当。。你就将就吧。 一句话，就是类似于通行证的密码。