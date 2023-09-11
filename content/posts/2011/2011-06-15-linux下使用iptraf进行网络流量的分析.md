---
title: Linux下使用Iptraf进行网络流量的分析
author: admin
type: post
date: 2011-06-15T02:45:17+00:00
url: /archives/9804
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Iptraf
 - Linux
 - 流量监控

---
下面的教程我个人安装的时候,总是失败,在/usr/local/bin目录里没有iptraf这个文件,没有办法直接用

> **yum -y install iptraf**

命令安装成功了.

Iptraf是一款Linux环境下，监控网络流量的一款绝佳的免费小软件，特别是安装到防火墙上，与Iptables一起工作，监控流经防火墙的网络异常，效果非常好。

我的安装配置环境是redhat 9.0

**一、软件下载**

Iptraf的最新版本是2.7.0，可以从下面的地址下载[][1]

**二、安装环境需要**

— gcc 2.7.2.3 or later

— GNU C (glibc) development library 2.1 or later

— ncurses development libraries 4.2 or later

可以在Linux下执行：

> \# rpm -qa | grep gcc
>
> \# rpm -qa | grep glibc
>
> \# rpm -qa | grep ncurses

如果没有，则请安装。

**三、安装**

从网站下载软件包,将下载得到的Iptraf-2.7.0.tar.gz上传到你所要安装的机器上，我的是防火墙的 /home/yang/ 目录：

> \# cd /home/yang
>
> \# tar zxf Iptraf-2.7.0.tar.gz
>
> \# cd Iptraf-2.7.0
>
> \# ./Setup

至此，安装完毕。

安装程序会将执行程序安装到 /usr/local/bin 目录下，并创 /var/local/Iptraf 目录放置Iptraf的配置文件，同时创建 /var/log/Iptraf 目录放置Iptraf产生的日志文件。

**四、运行Iptraf**

确认环境变量的PATH变量包含路径 /usr/local/bin

\# Iptraf

运行Iptraf后会产生一个字符界面的菜单，点击x可以退出 Iptraf，各菜单说明如下：

1、菜单Configure…

在这里可以对 Iptraf 进行配置，所有的修改都将保存在文件：/var/local/Iptraf/Iptraf.cfg 中

— Reverse DNS Lookups 选项，对IP地址反查 DNS名，默认是关闭的。

— TCP/UDP Service Names 选项，使用服务器代替端口号，例如用www 代替80，默认是关闭的。

— Force promiscuous 混杂模式，此时网卡将接受所有到达的数据，不管是不是发给自己的。

— Color 终端显示彩色，当然用telnet ,ssh连接除外，也就是用不支持颜色的终端连接肯定还是没有颜色。

— Logging 同时产生日志文件，在/var/log/Iptraf 目录下。

— Activity mode 可以选择统计单位是kbit/sec 还是 kbyte/sec 。

— Source MAC addrs in traffic monitor 选择后，会显示数据包的源MAC地址。

2、菜单Filters…

在这里可以设置过滤规则，这是最有用的选项了，当你从远端连入监控机时，自己的机器与监控机会产生源源不断的tcp数据包，有时很令人讨厌，此时你就可以将自己的ip地址排除在外。

它包括六个选项，分别是：Tcp、Udp、Other IP、ARP、RARP、Non-ip。我们以TCP为例说明，其它选项的配置都很相似。

— Defining a New Filter

选择Defining a New Filter后，会出来一对话框，要求填入对所建的当前规则的描述名，然后回车确定，Ctrl＋x取消。再接着出现的对话框里，Host name/IP address:的First里面填源地址，Second里填目标地址，Wildcard mask的两个框里面分别是源地址和目标地址所对应的掩码，注意，这里的地址即可以是单个地址，也可以是一个网段，如果是单个IP，则相应的子网掩码要填成255.255.255.255，如果是一个网段，则填写相应的子网掩码：例如，想表示192.168.0.0，有256个IP地址的网段，则填写192.168.0.0，子网是：255.255.255.0，其它类推，All则用0.0.0.0，子网也是0.0.0.0表示。

Port：栏要求填入要过滤的端口号，0表示任意端口号。Include/Exclude栏要求填入I或者E，I表示包括，E表示排除。填写完毕，回车确认，Ctrl x取消。

— Applying a Filter

我们在上一步定义的一个或多个过滤规则会存储为一个过滤列表，在没有应用之前并不起作用，我们可以在这里选择我们应用那些过滤规则。所有应用的规则会一直起作用，即使重新启动Iptraf。我们可以执行Detaching a Filter来取消执行当前所有应用的规则。

— Editing a Defined Filter 编辑一个已经存在的规则

— Deleting a Defined Filter 删除一个已经定义的规则

— Detaching a Filter 取消执行当前所有应用的规则

3、菜单IP Traffic Monitor

IP数据包流量实时监控窗口，注意这里会监控所有的来往数据包，包括自己的，所以，如果你使用远程终端连接上来的话，你和监控机将会源源不断的产生数据流，因此建议在Filters…菜单中将自己的IP过滤掉，是它不产生影响。在这里可以实时的看到每一个连接的流量状态，它有两个窗口，上面的是TCP的连接状态，下面的窗口可以看到UDP、ICMP、OSPF、IGRP、IGP、IGMP、GRE、ARP、RARP的数据包。可以点击s键选择排序，可以按照包的数量排序，也可按照字节的大小排序，如果因为它是实时变化的而导致看不太清楚的话，可以在Configure菜单中把Logging功能打开，它就会在/var/log/Iptraf 目录中记录日志，以方便你在日后查看，当Logging功能打开后，当你开始监控Iptraffic时，程序会提示你输入Log文件的文件名，默认的是ip_traffic-1.log。

在一个比较繁忙的网络里，显示的结果可能很乱，以至于你很难找到自己感兴趣的数据，这时可以使用Filters菜单，来过滤显示的数据。

4、菜单General Interface Statistics

这里显示每个网络设备出去和进入的数据流量统计信息，包括总计、IP包、非IP包、Bad IP包、还有每秒的流速，单位是kbit/sec或者是kbyte/sec ，这由Configure菜单的Activity选项决定。

如果设置了Filter选项，这里也受到影响。

5、菜单Detailed Interface Statistics

这里包括了每个网络设备的详细的统计信息，很简单，不再赘述。

6、Statistical Breakdowns

这里提供更详细的统计信息，可以按包的大小分类，分别统计；也可以按Tcp/Udp的服务来分类统计，也不再赘述。

7、LAN Station Statistics

提供对每个网络地址通过本机的数据的统计信息。



 [1]: ftp://iptraf.seul.org/pub/Iptraf/