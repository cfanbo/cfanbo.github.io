---
title: Centos5下配置 lvs DR
author: admin
type: post
date: 2010-10-26T07:49:37+00:00
url: /archives/6381
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - centos
 - ipvsadm
 - lvs

---
系统环境如下:

server1:192.168.1.206 vip server centos5
server2:192.168.1.210 apache centos5
server3:192.168.1.211 apache centos5

vip:192.168.1.208
port:80

============================================

下面的安装是在vip server上进行的

1、查看自己的操作系统的内核 #uname -a

2、这个内核已经包括了ipvs的补丁,进行如下的操作就可以

3、下面建立一个指向,为了保证ipvsadm安装

1. modprobe ip_vs

2. cat /proc/net/ip_vs


出现如下的提示

IP Virtual Server version 1.2.1 (size=4096)Prot LocalAddress:Port Scheduler Flags

– > RemoteAddress:Port Forward Weight ActiveConn InActConn

**4、安装ipvsadm**

[http://www.linuxvirtualserver.org/software/kernel-2.6/ipvsadm-1.24.tar.gz](http://www.linuxvirtualserver.org/software/kernel-2.6/ipvsadm-1.24.tar.gz)

下载吧

> tar -xzvf ipvsadm-1.24.tar.gzcd ipvsadm-1.24
>
> make && make install

如果出现错误:

> libipvs.c:309: error: dereferencing pointer to incomplete typelibipvs.c:315: error: `IP_VS_SO_GET_DAEMON’ undeclared (first use in this function)
>
> libipvs.c: At top level:
>
>
> libipvs.c:33: error: storage size of `ipvs_info’ isn’t known
>
>
> make[1]: *** [libipvs.o] Error 1
>
>
> make[1]: Leaving directory `/linlan/ipvsadm-1.24/libipvs’
>
>
> make: *** [libs] Error 2

解决方法是:

>

> ln -s /usr/src/kernels/2.6.18-92.el5-i686/ /usr/src/linux
>

/usr/src/kernels/2.6.18-92.el5-i686/ 是你自己的内核源码路径

再执行

> make && make install

安装完成后可以通过ipvsadm命令查看是否安装成功

IP Virtual Server version 1.2.1 (size=4096)Prot LocalAddress:Port Scheduler Flags

– > RemoteAddress:Port           Forward Weight ActiveConn InActConn

接下来配置VIP服务器

**5、配置VIP脚本**

均衡器 代码

下载: [vip.sh](http://xok.la/wp-content/plugins/coolcode/coolcode.php?p=313&download=vip.sh)

> #!/bin/sh# description: start LVS   of  Directorserver
>
> VIP=192.168.1.208
>
> RIP1=192.168.1.210
>
> RIP2=192.168.1.211
>
>
> #RIPn=192.168.1.128~254
>
>
> GW=192.168.1.1
>
>
> . /etc/rc.d/init.d/functions
>
>
> case “$1″ in
>
>
> start)
>
>
> echo ” start LVS  of DirectorServer”
>
>
> # set the Virtual  IP Address
>
> /sbin/ifconfig eth0:0 $VIP broadcast $VIP netmask 255.255.255.255 up
>
> /sbin/route add -host $VIP dev eth0:0
>
>
> #Clear IPVS table
>
> /sbin/ipvsadm -C
>
>
> #set LVS
>
> /sbin/ipvsadm -A -t $VIP:80 -s rr
>
> /sbin/ipvsadm -a -t $VIP:80 -r $RIP1:80 -g
>
> /sbin/ipvsadm -a -t $VIP:80 -r $RIP2:80 -g
>
>
> #Run LVS
>
> /sbin/ipvsadm
>
> #end
>
>
> ;;
>
>
> stop)
>
>
> echo “close LVS Directorserver”
>
>
> /sbin/ipvsadm -C
>
>
> ;;
>
>
> *)
>
>
> echo “Usage: $0 {start|stop}”
>
>
> exit 1
>
>
> esac

接下来配置realserver

**6、配置realserver脚本**

下载: [realserver.sh](http://xok.la/wp-content/plugins/coolcode/coolcode.php?p=313&download=realserver.sh)

> #!/bin/bash#description : start realserver
>
> VIP=192.168.1.208
>
>
> /sbin/ifconfig lo:0 $VIP broadcast $VIP netmask 255.255.255.255 up
>
>
> /sbin/route add -host $VIP dev lo:0
>
>
> echo “1” >/proc/sys/net/ipv4/conf/lo/arp_ignore
>
>
> echo “2” >/proc/sys/net/ipv4/conf/lo/arp_announce
>
>
> echo “1” >/proc/sys/net/ipv4/conf/all/arp_ignore
>
>
> echo “2” >/proc/sys/net/ipv4/conf/all/arp_announce
>
>
> sysctl -p
>
>
> #end

分别在两台realserver上运行该脚本,然后重新启动apache,至此配置已经完成了,下面来看看测试的过程

首先在vip server上打开控制台,你会看见lvs的列表:

> [root@xok linlan]# ipvsadm -LnIP Virtual Server version 1.2.1 (size=4096)
>
> Prot LocalAddress:Port Scheduler Flags
>
> – > RemoteAddress:Port           Forward Weight ActiveConn InActConn
>
>
> TCP  192.168.1.208:80 rr
>
>
> – > 192.168.1.211:80             Route   1      0          0
>
>
> – > 192.168.1.210:80             Route   1      0          0

从这里我们可以看到有两台realserver在后台提供转发后的访问,打开浏览器,输入http://192.168.1.208,浏览器会返回你访问的web的结果,在打开另外一个浏览器输入同样的地址,返回同样的结果,然后回到控制台看看情况:

> [root@xok linlan]# ipvsadm -LnIP Virtual Server version 1.2.1 (size=4096)
>
> Prot LocalAddress:Port Scheduler Flags
>
> – > RemoteAddress:Port           Forward Weight ActiveConn InActConn
>
>
> TCP  192.168.1.208:80 rr
>
>
> – > 192.168.1.211:80             Route   1    0          10
>
>
> – > 192.168.1.210:80             Route   1    0          10

发现InActConn变成了10,表示两个服务器都接收到了转发,同时还可以打开apache的log,会发现刚才的web访问已经发送到两台realserver了,表明配置成功了！

**ipvsadm命令**

下面看看lvs控制台的基本命令

添加一个Service

> - # ipvsadm -A -t 192.168.1.208:80 -s rr
>
> - rr:表示轮询的方法,缺省为wcl,方法有rr|wrr|lc|wlc|lblc|lblcr|dh|sh|sed|nq这些.

rr:轮叫(Round Robin)

调度器通过”轮叫”调度算法将外部请求按顺序轮流分配到集群中的真实服务器上，它均等地对待每一台服务器，而不管服务器上实际的连接数和系统负载.

wrr:加权轮叫(Weighted Round Robin)

调度器通过”加权轮叫”调度算法根据真实服务器的不同处理能力来调度访问请求.这样可以保证处理能力强的服务器处理更多的访问流量.调度器可以自动问询真实服务器的负载情况，并动态地调整其权值.

lc:最少链接(Least Connections)

调度器通过”最少连接”调度算法动态地将网络请求调度到已建立的链接数最少的服务器上.如果集群系统的真实服务器具有相近的系统性能，采用”最小连接”调度算法可以较好地均衡负载.

wlc:加权最少链接(Weighted Least Connections)

在集群系统中的服务器性能差异较大的情况下，调度器采用”加权最少链接”调度算法优化负载均衡性能，具有较高权值的服务器将承受较大比例的活动连接负载.调度器可以自动问询真实服务器的负载情况，并动态地调整其权值.

lblc:基于局部性的最少链接(Locality-Based Least Connections)

“基于局部性的最少链接” 调度算法是针对目标IP地址的负载均衡，目前主要用于Cache集群系统.该算法根据请求的目标IP地址找出该目标IP地址最近使用的服务器，若该服务器是可用的且没有超载，将请求发送到该服务器；若服务器不存在，或者该服务器超载且有服务器处于一半的工作负载，则用”最少链接”的原则选出一个可用的服务器，将请求发送到该服务器.

lblcr:带复制的基于局部性最少链接(Locality-Based Least Connections with Replication)

“带复制的基于局部性最少链接”调度算法也是针对目标IP地址的负载均衡，目前主要用于Cache集群系统.它与LBLC算法的不同之处是它要维护从一个目标IP地址到一组服务器的映射，而LBLC算法维护从一个目标IP地址到一台服务器的映射.该算法根据请求的目标IP地址找出该目标IP地址对应的服务器组，按”最小连接”原则从服务器组中选出一台服务器，若服务器没有超载，将请求发送到该服务器，若服务器超载；则按”最小连接”原则从这个集群中选出一台服务器，将该服务器加入到服务器组中，将请求发送到该服务器.同时，当该服务器组有一段时间没有被修改，将最忙的服务器从服务器组中删除，以降低复制的程度.

dh:目标地址散列(Destination Hashing)

“目标地址散列”调度算法根据请求的目标IP地址，作为散列键(Hash Key)从静态分配的散列表找出对应的服务器，若该服务器是可用的且未超载，将请求发送到该服务器，否则返回空.

sh:源地址散列(Source Hashing)

“源地址散列”调度算法根据请求的源IP地址，作为散列键(Hash Key)从静态分配的散列表找出对应的服务器，若该服务器是可用的且未超载，将请求发送到该服务器，否则返回空.

sed|nq:暂无

添加一个realserver

> - # ipvsadm -a -t 192.168.1.208:80 -r 192.168.1.180:80 -g
>
> - -a:添加一个realserver
>
> - -r:realserver的地址
>
> - -g:缺省参数
>
> - -w:lvs转发通道的处理能力

> - 修改realserver
>
> - # ipvsadm -e -t 192.168.1.208:80 -r 192.168.1.180:80 -w 100
>
> - -e:修改-r参数的realserver

详细的参数:

[root@xok linlan]# ipvsadm -hipvsadm v1.24 2005/12/10 (compiled with popt and IPVS v1.2.1)

Usage:

ipvsadm -A|E -t|u|f service-address [-s scheduler] [-p [timeout]] [-M netmask]


ipvsadm -D -t|u|f service-address


ipvsadm -C


ipvsadm -R


ipvsadm -S [-n]


ipvsadm -a|e -t|u|f service-address -r server-address [options]


ipvsadm -d -t|u|f service-address -r server-address


ipvsadm -L|l [options]


ipvsadm -Z [-t|u|f service-address]


ipvsadm –set tcp tcpfin udp


ipvsadm –start-daemon state [–mcast-interface interface] [–syncid sid]


ipvsadm –stop-daemon state


ipvsadm -h


Commands:


Either long or short options are allowed.


–add-service     -A        add virtual service with options


–edit-service    -E        edit virtual service with options


–delete-service  -D        delete virtual service


–clear           -C        clear the whole table


–restore         -R        restore rules from stdin


–save            -S        save rules to stdout


–add-server      -a        add real server with options


–edit-server     -e        edit real server with options


–delete-server   -d        delete real server


–list            -L|-l     list the table


–zero            -Z        zero counters in a service or all services


–set tcp tcpfin udp        set connection timeout values


–start-daemon              start connection sync daemon


–stop-daemon               stop connection sync daemon


–help            -h        display this help message


Options:


–tcp-service  -t service-address   service-address is host[:port]


–udp-service  -u service-address   service-address is host[:port]


–fwmark-service  -f fwmark         fwmark is an integer greater than zero


–scheduler    -s scheduler         one of rr|wrr|lc|wlc|lblc|lblcr|dh|sh|sed|nq,


the default scheduler is wlc.


–persistent   -p [timeout]         persistent service


–netmask      -M netmask           persistent granularity mask


–real-server  -r server-address    server-address is host (and port)


–gatewaying   -g                   gatewaying (direct routing) (default)


–ipip         -i                   ipip encapsulation (tunneling)


–masquerading -m                   masquerading (NAT)


–weight       -w weight            capacity of real server


–u-threshold  -x uthreshold        upper threshold of connections


–l-threshold  -y lthreshold        lower threshold of connections


–mcast-interface interface         multicast interface for connection sync


–syncid sid                        syncid for connection sync (default=255)


–connection   -c                   output of current IPVS connections


–timeout                           output of timeout (tcp tcpfin udp)


–daemon                            output of daemon information


–stats                             output of statistics information


–rate                              output of rate information


–exact                             expand numbers (display exact values)


–thresholds                        output of thresholds information


–persistent-conn                   output of persistent connection info


–sort                              sorting output of service/server entries


–numeric      -n                   numeric output of addresses and ports


[root@xok linlan]#


ipvsadm命令中文参考

1,virtual-service-address:是指虚拟服务器的ip 地址2,real-service-address:是指真实服务器的ip 地址

3,scheduler:调度方法


ipvsadm 的用法和格式如下:


ipvsadm -A|E -t|u|f virutal-service-address:port [-s scheduler] [-p


[timeout]] [-M netmask]


ipvsadm -D -t|u|f virtual-service-address


ipvsadm -C


ipvsadm -R


ipvsadm -S [-n]


ipvsadm -a|e -t|u|f service-address:port -r real-server-address:port


[-g|i|m] [-w weight]


ipvsadm -d -t|u|f service-address -r server-address


ipvsadm -L|l [options]


ipvsadm -Z [-t|u|f service-address]


ipvsadm –set tcp tcpfin udp


ipvsadm –start-daemon state [–mcast-interface interface]


ipvsadm –stop-daemon


ipvsadm -h


**命令选项解释:**

有两种命令选项格式，长的和短的，具有相同的意思.在实际使用时，两种都可


以.


 -A –add-service 在内核的虚拟服务器表中添加一条新的虚拟服务器记录.也


就是增加一台新的虚拟服务器.


 -E –edit-service 编辑内核虚拟服务器表中的一条虚拟服务器记录.


 -D –delete-service 删除内核虚拟服务器表中的一条虚拟服务器记录.


 -C –clear 清除内核虚拟服务器表中的所有记录.


 -R –restore 恢复虚拟服务器规则


 -S –save 保存虚拟服务器规则，输出为-R 选项可读的格式


 -a –add-server 在内核虚拟服务器表的一条记录里添加一条新的真实服务器


记录.也就是在一个虚拟服务器中增加一台新的真实服务器


 -e –edit-server 编辑一条虚拟服务器记录中的某条真实服务器记录


 -d –delete-server 删除一条虚拟服务器记录中的某条真实服务器记录


 -L|-l –list 显示内核虚拟服务器表


 -Z –zero 虚拟服务表计数器清零（清空当前的连接数量等）


–set tcp tcpfin udp 设置连接超时值


–start-daemon 启动同步守护进程.他后面可以是master 或backup，用来说


明LVS Router 是master 或是backup.在这个功能上也可以采用keepalived 的


VRRP 功能.


–stop-daemon 停止同步守护进程


 -h –help 显示帮助信息


其他的选项:


 -t –tcp-service service-address 说明虚拟服务器提供的是tcp 的服务


[vip:port] or [real-server-ip:port]


 -u –udp-service service-address 说明虚拟服务器提供的是udp 的服务


[vip:port] or [real-server-ip:port]


 -f –fwmark-service fwmark 说明是经过iptables 标记过的服务类型.


 -s –scheduler scheduler 使用的调度算法，有这样几个选项


rr|wrr|lc|wlc|lblc|lblcr|dh|sh|sed|nq,


默认的调度算法是： wlc.


 -p –persistent [timeout] 持久稳固的服务.这个选项的意思是来自同一个客


户的多次请求，将被同一台真实的服务器处理.timeout 的默认值为300 秒.


 -M –netmask netmask persistent granularity mask


 -r –real-server server-address 真实的服务器[Real-Server:port]


 -g –gatewaying 指定LVS 的工作模式为直接路由模式（也是LVS 默认的模式）


 -i –ipip 指定LVS 的工作模式为隧道模式


 -m –masquerading 指定LVS 的工作模式为NAT 模式


 -w –weight weight 真实服务器的权值


–mcast-interface interface 指定组播的同步接口


 -c –connection 显示LVS 目前的连接 如：ipvsadm -L -c


–timeout 显示tcp tcpfin udp 的timeout 值 如：ipvsadm -L –timeout


–daemon 显示同步守护进程状态


–stats 显示统计信息


–rate 显示速率信息


–sort 对虚拟服务器和真实服务器排序输出


–numeric -n 输出IP 地址和端口的数字形式