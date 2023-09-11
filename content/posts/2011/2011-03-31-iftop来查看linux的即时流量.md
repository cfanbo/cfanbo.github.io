---
title: iftop来查看linux的即时流量
author: admin
type: post
date: 2011-03-31T13:07:18+00:00
url: /archives/8807
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - iftop
 - Linux
 - 流量监控

---
**关于 Iftop**

iftop 是类似于top的实时流量监控工具。主要用来显示本机网络流量情况及各相互通信的流量集合，如单独同那台机器间的流量大小，非常适合于代理服务器和iptables服务器使用

官方网站：

**安装iftop**
**安装方法1、编译安装
**
如果采用编译安装可以到iftop官网下载最新的源码包。

安装前需要已经安装好基本的编译所需的环境，比如make、gcc、autoconf等。安装iftop还需要安装libpcap和libcurses。

CentOS上安装所需依赖包：

> yum install flex byacc  libpcap ncurses ncurses-devel libpcap-devel

Debian上安装所需依赖包：

apt-get install flex byacc  libpcap0.8 libncurses5

下载iftop

```
wget http://www.ex-parrot.com/pdw/iftop/download/iftop-0.17.tar.gz
tar zxvf iftop-0.17.tar.gz
cd iftop-0.17
./configure

make && make install
```

configure: error: can’t find pcap.h
You’re not going to get very far without libpcap.
那你需要先安装libpcap，找到相应的rpm文件，比如：

-rw-r–r– 1 root root  108987 Apr  3 08:21 libpcap-0.9.4-8.1.i386.rpm
-rw-r–r– 1 root root  119062 Apr  3 08:21 libpcap-devel-0.9.4-8.1.i386.rpm

**安装方法2：(懒人办法，最简单)**
直接省略上面的步骤
flibpcap-0.9.4-14.el5.x86_64.rpm
CentOS系统(已经测试可以使用)：

> yum install flex byacc  libpcap ncurses ncurses-devel
>
> wget
>
> rpm -ivh iftop-0.17-1.el5.rf.i386.rpm

Debian系统 运行：apt-get install iftop

**运行iftop**
直接运行： iftop



**1、iftop界面相关说明**
界面上面显示的是类似刻度尺的刻度范围，为显示流量图形的长条作标尺用的。

中间的<= =>这两个左右箭头，表示的是流量的方向。

TX：发送流量
RX：接收流量
TOTAL：总流量
Cumm：运行iftop到目前时间的总流量
peak：流量峰值
rates：分别表示过去 2s 10s 40s 的平均流量

**2、iftop相关参数**
常用的参数
-i设定监测的网卡，如：# iftop -i eth1

-B 以bytes为单位显示流量(默认是bits)，如：# iftop -B

-n使host信息默认直接都显示IP，如：# iftop -n

-N使端口信息默认直接都显示端口号，如: # iftop -N

-F显示特定网段的进出流量，如# iftop -F 10.10.1.0/24或# iftop -F 10.10.1.0/255.255.255.0

-h（display this message），帮助，显示参数信息

-p使用这个参数后，中间的列表显示的本地主机信息，出现了本机以外的IP信息;

-b使流量图形条默认就显示;

-f这个暂时还不太会用，过滤计算包用的;

-P使host信息及端口信息默认就都显示;

-m设置界面最上边的刻度的最大值，刻度分五个大段显示，例：# iftop -m 100M

**进入iftop画面后的一些操作命令(注意大小写)**

按h切换是否显示帮助;

按n切换显示本机的IP或主机名;

按s切换是否显示本机的host信息;

按d切换是否显示远端目标主机的host信息;

按t切换显示格式为2行/1行/只显示发送流量/只显示接收流量;

按N切换显示端口号或端口服务名称;

按S切换是否显示本机的端口信息;

按D切换是否显示远端目标主机的端口信息;

按p切换是否显示端口信息;

按P切换暂停/继续显示;

按b切换是否显示平均流量图形条;

按B切换计算2秒或10秒或40秒内的平均流量;

按T切换是否显示每个连接的总流量;

按l打开屏幕过滤功能，输入要过滤的字符，比如ip,按回车后，屏幕就只显示这个IP相关的流量信息;

按L切换显示画面上边的刻度;刻度不同，流量图形条会有变化;

按j或按k可以向上或向下滚动屏幕显示的连接记录;

按1或2或3可以根据右侧显示的三列流量数据进行排序;

按<根据左边的本机名或IP排序;

按>根据远端目标主机的主机名或IP排序;

按o切换是否固定只显示当前的连接;

按f可以编辑过滤代码，这是翻译过来的说法，我还没用过这个！

按!可以使用shell命令，这个没用过！没搞明白啥命令在这好用呢！

按q退出监控。