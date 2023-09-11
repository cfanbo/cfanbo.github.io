---
title: linux tcpdump命令参数及用法详解–linux下抓包网络分析
author: admin
type: post
date: 2012-04-29T16:58:00+00:00
url: /archives/12815
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - tcpdump

---
采用命令行方式，它的命令格式为：

**tcpdump \[ -adeflnNOpqStvx \] \[ -c 数量 \] \[ -F 文件名 \]\[ -i 网络接口 \] \[ -r 文件名\] \[ -s snaplen \]\[ -T 类型 \] \[ -w 文件名 \] [表达式 ]**
****

> **1. tcpdump的选项介绍**
> -a 　　　将网络地址和广播地址转变成名字；
> -d 　　　将匹配信息包的代码以人们能够理解的汇编格式给出；
> -dd 　　　将匹配信息包的代码以c语言程序段的格式给出；
> -ddd 　　　将匹配信息包的代码以十进制的形式给出；
> -e 　　　在输出行打印出数据链路层的头部信息；
> -f 　　　将外部的Internet地址以数字的形式打印出来；
> -l 　　　使标准输出变为缓冲行形式；
> -n 　　　不把网络地址转换成名字；
> -t 　　　在输出的每一行不打印时间戳；
> -v 　　　输出一个稍微详细的信息，例如在ip包中可以包括ttl和服务类型的信息；
> -vv 　　　输出详细的报文信息；
> -c 　　　在收到指定的包的数目后，tcpdump就会停止；
> -F 　　　从指定的文件中读取表达式,忽略其它的表达式；
> -i 　　　指定监听的网络接口；
> -r 　　　从指定的文件中读取包(这些包一般通过-w选项产生)；
> -w 　　　直接将包写入文件中，并不分析和打印出来；
> -T 　　　将监听到的包直接解释为指定的类型的报文，常见的类型有rpc （远程过程调用）和snmp（简单网络管理协议；）

第一种是关于类型的关键字，主要包括host，net，port, 例如 host 210.27.48.2，指明 210.27.48.2是一台主 机，net 202.0.0.0 指明 202.0.0.0是一个网络地址，port 23 指明端口号是23。如果没有指定类型，缺省的类型是 host.


第二种是确定传输方向的关键字，主要包括src , dst ,dst or src, dst and src ,这些关键字指明了传输的方向。举例说 明，src 210.27.48.2 ,指明ip包中源地址是210.27.48.2 , dst net 202.0.0.0 指明目的网络地址是 202.0.0.0 。如果没有指明方向关键字，则缺省是src or dst关键字。
第三种是协议的关键字，主要包括fddi,ip,arp,rarp,tcp,udp等类型。Fddi指明是在FDDI(分布式光纤数据接口网络)上的特定的网络协议，实际上它是 “ether”的alias.html’ target=’_blank’>别名，fddi和ether具有类似的源地址和目的地址，所以可以将fddi协议包当作ether的包进行处理和分析。其他的几个关键字就是指明了监听的包的协议内容。如果没有指定任何协议，则tcpdump将会监听所有协议的信息包。
除了这三种类型的关键字之外，其他重要的关键字如下：gateway, broadcast,less,greater,还有三种逻辑运算，取非运算是 ‘not ‘ ‘! ‘, 与运算是’and’,’&&’;或运算 是’or’ ,’││’；这些关键字可以组合起来构成强大的组合条件来满足人们的需要，下面举几个例子来说明。
普通情况下，直接启动tcpdump将监视第一个网络界面上所有流过的数据包。

> # tcpdump
> tcpdump: listening on fxp0
> 11:58:47.873028 202.102.245.40.netbios-ns > 202.102.245.127.netbios-ns: udp 50
> 11:58:47.974331 0:10:7b:8:3a:56 > 1:80:c2:0:0:0 802.1d ui/C len=43
> 0000 0000 0080 0000 1007 cf08 0900 0000
> 0e80 0000 902b 4695 0980 8701 0014 0002
> 000f 0000 902b 4695 0008 00
> 11:58:48.373134 0:0:e8:5b:6d:85 > Broadcast sap e0 ui/C len=97
> ffff 0060 0004 ffff ffff ffff ffff ffff
> 0452 ffff ffff 0000 e85b 6d85 4008 0002
> 0640 4d41 5354 4552 5f57 4542 0000 0000
> 0000 00

使用-i参数指定tcpdump监听的网络界面，这在计算机具有多个网络界面时非常有用，
使用-c参数指定要监听的数据包数量，
使用-w参数指定将监听到的数据包写入文件中保存
A想要截获所有210.27.48.1 的主机收到的和发出的所有的数据包：
#tcpdump host 210.27.48.1
B想要截获主机210.27.48.1 和主机210.27.48.2 或210.27.48.3的通信，使用命令：（在命令行中适用　　　括号时，一定要
#tcpdump host 210.27.48.1 and (210.27.48.2 or 210.27.48.3 )
C如果想要获取主机210.27.48.1除了和主机210.27.48.2之外所有主机通信的ip包，使用命令：
#tcpdump ip host 210.27.48.1 and ! 210.27.48.2
D如果想要获取主机210.27.48.1接收或发出的telnet包，使用如下命令：
#tcpdump tcp port 23 host 210.27.48.1
E 对本机的udp 123 端口进行监视 123 为ntp的服务端口
\# tcpdump udp port 123

F 系统将只对名为hostname的主机的通信数据包进行监视。主机名可以是本地主机，也可以是网络上的任何一台计算机。下面的命令可以读取主机hostname发送的所有数据：
#tcpdump -i eth0 src host hostname
G 下面的命令可以监视所有送到主机hostname的数据包：
#tcpdump -i eth0 dst host hostname
H 我们还可以监视通过指定网关的数据包：
#tcpdump -i eth0 gateway Gatewayname
I 如果你还想监视编址到指定端口的TCP或UDP数据包，那么执行以下命令：
#tcpdump -i eth0 host hostname and port 80
J 如果想要获取主机210.27.48.1除了和主机210.27.48.2之外所有主机通信的ip包
，使用命令：
#tcpdump ip host 210.27.48.1 and ! 210.27.48.2
K 想要截获主机210.27.48.1 和主机210.27.48.2 或210.27.48.3的通信，使用命令
：（在命令行中适用　　　括号时，一定要
#tcpdump host 210.27.48.1 and (210.27.48.2 or 210.27.48.3 )
L 如果想要获取主机210.27.48.1除了和主机210.27.48.2之外所有主机通信的ip包，使用命令：
#tcpdump ip host 210.27.48.1 and ! 210.27.48.2
M 如果想要获取主机210.27.48.1接收或发出的telnet包，使用如下命令：
#tcpdump tcp port 23 host 210.27.48.1
第三种是协议的关键字，主要包括fddi,ip ,arp,rarp,tcp,udp等类型
除了这三种类型的关键字之外，其他重要的关键字如下：gateway, broadcast,less,greater,还有三种逻辑运算，取非运算是 ‘not ‘ ‘! ‘, 与运算是’and’,’&&’;或运算 是’or’ ,’||’；
第二种是确定传输方向的关键字，主要包括src , dst ,dst or src, dst and src ,如果我们只需要列出送到80端口的数据包，用dst port；如果我们只希望看到返回80端口的数据包，用src port。
#tcpdump –i eth0 host hostname and dst port 80 目的端口是80
或者
#tcpdump –i eth0 host hostname and src port 80 源端口是80 一般是提供http的服务的主机
如果条件很多的话 要在条件之前加and 或 or 或 not
#tcpdump -i eth0 host ! 211.161.223.70 and ! 211.161.223.71 and dst port 80
如果在ethernet 使用混杂模式 系统的日志将会记录
May 7 20:03:46 localhost kernel: eth0: Promiscuous mode enabled.
May 7 20:03:46 localhost kernel: device eth0 entered promiscuous mode
May 7 20:03:57 localhost kernel: device eth0 left promiscuous mode
tcpdump 对截获的数据并没有进行彻底解码，数据包内的大部分内容是使用十六进制的形式直接打印输出的。显然这不利于分析网络故障，通常的解决办法 是先使用带-w参数的tcpdump 截获数据并保存到文件中，然后再使用其他程序进行解码分析。当然也应该定义过滤规则，以避免捕获的数据包填满整个硬 盘。

TCPDUMP简介
在传统的网络分析和测试技术中，嗅探器(sniffer)是最常见，也是最重要的技术之一。sniffer工具首先是为网络管理员和网络程序员进行网 络分析而设计的。对于网络管理人员来说，使用嗅探器可以随时掌握网络的实际情况，在网络性能急剧下降的时候，可以通过sniffer工具来分析原因，找出 造成网络阻塞的来源。对于网络程序员来说,通过sniffer工具来调试程序。

用过windows平台上的sniffer工具(例如，netxray和sniffer pro软件)的朋友可能都知道，在共享式的局域网中，采用sniffer工具简直可以对网络中的所有流量一览无余！Sniffer工具实际上就是一个网络 上的抓包工具，同时还可以对抓到的包进行分析。由于在共享式的网络中，信息包是会广播到网络中所有主机的网络接口，只不过在没有使用sniffer工具之 前，主机的网络设备会判断该信息包是否应该接收，这样它就会抛弃不应该接收的信息包，sniffer工具却使主机的网络设备接收所有到达的信息包，这样就 达到了网络监听的效果。Linux作为网络服务器，特别是作为路由器和网关时，数据的采集和分析是必不可少的。所以，今天我们就来看看Linux中强大的网络数据采集分析工具——TcpDump。

用简单的话来定义tcpdump，就是：dump the traffice on a network，根据使用者的定义对网络上的数据包进行截获的包分析工具。

作为互联网上经典的的系统管理员必备工具，tcpdump以其强大的功能，灵活的截取策略，成为每个高级的系统管理员分析网络，排查问题等所必备的东东之一。

顾名思义，TcpDump可以将网络中传送的数据包的“头”完全截获下来提供分析。它支持针对网络层、协议、主机、网络或端口的过滤，并提供and、or、not等逻辑语句来帮助你去掉无用的信息。

tcpdump提供了源代码，公开了接口，因此具备很强的可扩展性，对于网络维护和入侵者都是非常有用的工具。tcpdump存在于基本的 FreeBSD系统中，由于它需要将网络界面设置为混杂模式，普通用户不能正常执行，但具备root权限的用户可以直接执行它来获取网络上的信息。因此系 统中存在网络分析工具主要不是对本机安全的威胁，而是对网络上的其他计算机的安全存在威胁。

普通情况下，直接启动tcpdump将监视第一个网络界面上所有流过的数据包。

－－－－－－－－－－－－－－－－－－－－－－－

> bash-2.02# tcpdump
>
> tcpdump: listening on eth0
>
> 11:58:47.873028 202.102.245.40.netbios-ns > 202.102.245.127.netbios-ns: udp 50
>
> 11:58:47.974331 0:10:7b:8:3a:56 > 1:80:c2:0:0:0 802.1d ui/C len=43
>
> 0000 0000 0080 0000 1007 cf08 0900 0000
>
> 0e80 0000 902b 4695 0980 8701 0014 0002
>
> 000f 0000 902b 4695 0008 00
>
> 11:58:48.373134 0:0:e8:5b:6d:85 > Broadcast sap e0 ui/C len=97
>
> ffff 0060 0004 ffff ffff ffff ffff ffff
>
> 0452 ffff ffff 0000 e85b 6d85 4008 0002
>
> 0640 4d41 5354 4552 5f57 4542 0000 0000
>
> 0000 00
>
> ^C

－－－－－－－－－－－－－－－－－－－－－－－－

首先我们注意一下，从上面的输出结果上可以看出来，基本上tcpdump总的的输出格式为：系统时间 来源主机.端口 > 目标主机.端口 数据包参数

TcpDump的参数化支持

tcpdump支持相当多的不同参数，如使用-i参数指定tcpdump监听的网络界面，这在计算机具有多个网络界面时非常有用，使用-c参数指定要监听的数据包数量，使用-w参数指定将监听到的数据包写入文件中保存，等等。

然而更复杂的tcpdump参数是用于过滤目的，这是因为网络中流量很大，如果不加分辨将所有的数据包都截留下来，数据量太大，反而不容易发现需要的 数据包。使用这些参数定义的过滤规则可以截留特定的数据包，以缩小目标，才能更好的分析网络中存在的问题。tcpdump使用参数指定要监视数据包的类 型、地址、端口等，根据具体的网络问题，充分利用这些过滤规则就能达到迅速定位故障的目的。请使用man tcpdump查看这些过滤规则的具体用法。

显然为了安全起见，不用作网络管理用途的计算机上不应该运行这一类的网络分析软件，为了屏蔽它们，可以屏蔽内核中的bpfilter伪设备。一般情况 下网络硬件和TCP/IP堆栈不支持接收或发送与本计算机无关的数据包，为了接收这些数据包，就必须使用网卡的混杂模式，并绕过标准的TCP/IP 堆栈才行。在FreeBSD下，这就需要内核支持伪设备bpfilter。因此，在内核中取消bpfilter支持，就能屏蔽tcpdump之类的网络分 析工具。

并且当网卡被设置为混杂模式时，系统会在控制台和日志文件中留下记录，提醒管理员留意这台系统是否被用作攻击同网络的其他计算机的跳板。

May 15 16:27:20 host1 /kernel: fxp0: promiscuous mode enabled

虽然网络分析工具能将网络中传送的数据记录下来，但是网络中的数据流量相当大，如何对这些数据进行分析、分类统计、发现并报告错误却是更关键的问题。 网络中的数据包属于不同的协议，而不同协议数据包的格式也不同。因此对捕获的数据进行解码，将包中的信息尽可能的展示出来，对于协议分析工具来讲更为重 要。昂贵的商业分析工具的优势就在于它们能支持很多种类的应用层协议，而不仅仅只支持tcp、udp等低层协议。

从上面tcpdump的输出可以看出，tcpdump对截获的数据并没有进行彻底解码，数据包内的大部分内容是使用十六进制的形式直接打印输出的。显 然这不利于分析网络故障，通常的解决办法是先使用带-w参数的tcpdump 截获数据并保存到文件中，然后再使用其他程序进行解码分析。当然也应该定义过滤规则，以避免捕获的数据包填满整个硬盘。

**TCP功能**

**数据过滤**

不带任何参数的TcpDump将搜索系统中所有的网络接口，并显示它截获的所有数据，这些数据对我们不一定全都需要，而且数据太多不利于分析。所以，我们应当先想好需要哪些数据，TcpDump提供以下参数供我们选择数据：

-b 在数据-链路层上选择协议，包括ip、arp、rarp、ipx都是这一层的。

例如：tcpdump -b arp 将只显示网络中的arp即地址转换协议信息。

-i 选择过滤的网络接口，如果是作为路由器至少有两个网络接口，通过这个选项，就可以只过滤指定的接口上通过的数据。例如：

tcpdump -i eth0 只显示通过eth0接口上的所有报头。

src、dst、port、host、net、ether、gateway这几个选项又分别包含src、dst 、port、host、net、ehost等附加选项。他们用来分辨数据包的来源和去向，src host 192.168.0.1指定源主机IP地址是192.168.0.1，dst net 192.168.0.0/24指定目标是网络192.168.0.0。以此类推，host是与其指定主机相关无论它是源还是目的，net是与其指定网络相 关的，ether后面跟的不是IP地址而是物理地址，而gateway则用于网关主机。可能有点复杂，看下面例子就知道了：

tcpdump src host 192.168.0.1 and dst net 192.168.0.0/24

过滤的是源主机为192.168.0.1与目的网络为192.168.0.0的报头。

tcpdump ether src 00:50:04:BA:9B and dst……

过滤源主机物理地址为XXX的报头（为什么ether src后面没有host或者net？物理地址当然不可能有网络喽）。

Tcpdump src host 192.168.0.1 and dst port not telnet

过滤源主机192.168.0.1和目的端口不是telnet的报头。

ip icmp arp rarp 和 tcp、udp、icmp这些选项等都要放到第一个参数的位置，用来过滤数据报的类型。

例如：

tcpdump ip src……

只过滤数据-链路层上的IP报头。

tcpdump udp and src host 192.168.0.1

只过滤源主机192.168.0.1的所有udp报头。

数据显示/输入输出

TcpDump提供了足够的参数来让我们选择如何处理得到的数据，如下所示：

-l 可以将数据重定向。

如tcpdump -l ＞tcpcap.txt将得到的数据存入tcpcap.txt文件中。

-n 不进行IP地址到主机名的转换。

如果不使用这一项，当系统中存在某一主机的主机名时，TcpDump会把IP地址转换为主机名显示，就像这样：eth0 ＜ ntc9.1165＞ router.domain.net.telnet，使用-n后变成了：eth0 ＜ 192.168.0.9.1165 ＞ 192.168.0.1.telnet。

-nn 不进行端口名称的转换。

上面这条信息使用-nn后就变成了：eth0 ＜ ntc9.1165 ＞ router.domain.net.23。

-N 不打印出默认的域名。

还是这条信息-N 后就是：eth0 ＜ ntc9.1165 ＞ router.telnet。

-O 不进行匹配代码的优化。

-t 不打印UNIX时间戳，也就是不显示时间。

-tt 打印原始的、未格式化过的时间。

-v 详细的输出，也就比普通的多了个TTL和服务类型。

［expression]的用法：

expression 是tcpdump最为有用的高级用法，可以利用它来匹配一些特殊的包。下面介绍一下expression的用法，主要是如何写出符合要求最为严格 expression。如果tcpdump中没有expression,那么tcpdump会把网卡上的所有数据包输出，否则会将被expression 匹配的包输出。

expression 由一个或多个[primitives]组成，而[primitives]由一个或多个[qualitifer]加一个id(name)或数字组成，它们的结构如用正则表达式则可表示为：

expression = ([qualitifer］+(id|number))+

依次看来，expression是一个复杂的条件表达式，其中[qualitifer］+(id|number)就是一个比较基本条件，qualitifer就表达一些的名称（项，变量），id或number则表示一个值（或常量）。

qualitifer共有三种，分别是：

type 表示id name或number涉及到的类型，这些词有host, nest, port ,portrange等等。

例子：

host foo 此为一个简单的primitive，host为qualitifer, foo为id name

net 128.3 net为qualitifer, 128.3为number

port 20

等等

每个privimtive必须有一个type词，如果表达式中没有，则默认是host.

dir 指定数据传输的方向，这些词有src, dst, src or dst, src and dst

例子：

dst net 128.3 ;此为一个相对复杂的primitive,结构为dir type number,表示目标网络为128.3的条件。
src or dst port ftp-data 此为比上一个相对简的结构，src or dst表示源或目标，ftp-data为id，表示ftp协议中数据传输端口，故整体表示源或目标端口ftp-data的数据包即匹配。

如果在一个primitive中没有dir词，此默认为src or dst. 如 host foo则表示源或目标主机为foo的数据包都匹配。

proto 此种词是用来匹配某种特定协议的，这些词包括：ether, fddi, tr, wlan, ip, ip6, arp, rarp, decnet,tcp和udp。其实这些词经常用来匹配某种协议，是使用率最高的一组词了。

上面三种qualitifer和id name或number组成一个primitive通常是下面这种方式的：

proto dir type id(number) ，即primitive=proto dir type (id | number)

如：

tcp src port 80

ip dst host 192.168.1.1

如果出现type的话，一定会出现id或num

如果出现dir，那么也会出现type,如果不出现，默认为host

而proto可单独出现，如 tcpdump ‘tcp’

通过上面介绍的三种qualitifer，我们很快就可以写出一个primitive，下面我就只用一个primitive作为expression匹配数据包。

(1)匹配ether包

匹配特定mac地址的数据包。

tcpdump ‘ether src 00:19:21:1D:75:E6’

匹配源mac为00:19:21:1D:75:E6的数据包其中src可改为dst, src or dst来匹改变条件

匹配ether广播包。ether广播包的特征是mac全1.故如下即可匹配：

tcpdump ‘ether dst ff:ff:ff:ff:ff:ff’

ylin@ylin:~$ sudo tcpdump -c 1 ‘ether dst ff:ff:ff:ff:ff:ff’

tcpdump: verbose output suppressed, use -v or -vv for full protocol decode

listening on eth0, link-type EN10MB (Ethernet), capture size 96 bytes

10:47:57.784099 arp who-has 192.168.240.77 tell 192.168.240.189

在此，只匹配1个包就退出了。第一个是arp请求包，arp请求包的是采用广播的方式发送的，被匹配那是当之无愧的。

匹配ether组播包，ether的组播包的特征是mac的最高位为1，其它位用来表示组播组编号，如果你想匹配其的多播组，知道它的组MAC地址即可。如

tcpdump ‘ether dst’ Mac_Address表示地址，填上适当的即可。如果想匹配所有的ether多播数据包，那么暂时请放下，下面会继续为你讲解更高级的应用。

(2)匹配arp包

arp包用于IP到Mac址转换的一种协议，包括arp请求和arp答应两种报文，arp请求报文是ether广播方式发送出去的，也即 arp请求报文的mac地址是全1，因此用ether dst FF;FF;FF;FF;FF;FF可以匹配arp请求报文，但不能匹配答应报文。因此要匹配arp的通信过程，则只有使用arp来指定协议。

tcpdump ‘arp’ 即可匹配网络上arp报文。

ylin@ylin:~$ arping -c 4 192.168.240.1>/dev/null& sudo tcpdump -p ‘arp’

[1] 9293

WARNING: interface is ignored: Operation not permitted

tcpdump: verbose output suppressed, use -v or -vv for full protocol decode

listening on eth0, link-type EN10MB (Ethernet), capture size 96 bytes

11:09:25.042479 arp who-has 192.168.240.1 (00:03:d2:20:04:28 (oui Unknown)) tell ylin.local

11:09:25.042702 arp reply 192.168.240.1 is-at 00:03:d2:20:04:28 (oui Unknown)

11:09:26.050452 arp who-has 192.168.240.1 (00:03:d2:20:04:28 (oui Unknown)) tell ylin.local

11:09:26.050765 arp reply 192.168.240.1 is-at 00:03:d2:20:04:28 (oui Unknown)

11:09:27.058459 arp who-has 192.168.240.1 (00:03:d2:20:04:28 (oui Unknown)) tell ylin.local

11:09:27.058701 arp reply 192.168.240.1 is-at 00:03:d2:20:04:28 (oui Unknown)

11:09:33.646514 arp who-has ylin.local tell 192.168.240.1

11:09:33.646532 arp reply ylin.local is-at 00:19:21:1d:75:e6 (oui Unknown)

本例中使用arping -c 4 192.168.240.1产生arp请求和接收答应报文，而tcpdump -p ‘arp’匹配出来了。此处-p选项是使网络工作于正常模式（非混杂模式），这样是方便查看匹配结果。

(3)匹配IP包

众所周知，IP协议是TCP/IP协议中最重要的协议之一，正是因为它才能把Internet互联起来，它可谓功不可没，下面分析匹配IP包的表达式。

对IP进行匹配

tcpdump ‘ip src 192.168.240.69’

ylin@ylin:~$ sudo tcpdump -c 3 ‘ip src 192.168.240.69’

tcpdump: verbose output suppressed, use -v or -vv for full protocol decode

listening on eth0, link-type EN10MB (Ethernet), capture size 96 bytes

11:20:00.973605 IP ylin.local.51486 > walnut.crossbeamsys.com.ssh: S 2706301341:2706301341(0) win 5840

11:20:00.974328 IP ylin.local.32849 > 192.168.200.150.domain: 5858+ PTR? 20.200.168.192.in-addr.arpa. (45)

11:20:01.243490 IP ylin.local.51486 > walnut.crossbeamsys.com.ssh: . ack 2762262674 win 183

IP广播组播数据包匹配：只需指明广播或组播地址即可

tcpdump ‘ip dst 240.168.240.255’

ylin@ylin:~$ sudo tcpdump ‘ip dst 192.168.240.255’

tcpdump: verbose output suppressed, use -v or -vv for full protocol decode

listening on eth0, link-type EN10MB (Ethernet), capture size 96 bytes

11:25:29.690658 IP dd.local > 192.168.240.255: ICMP echo request, id 10022, seq 1, length 64

11:25:30.694989 IP dd.local > 192.168.240.255: ICMP echo request, id 10022, seq 2, length 64

11:25:31.697954 IP dd.local > 192.168.240.255: ICMP echo request, id 10022, seq 3, length 64

11:25:32.697970 IP dd.local > 192.168.240.255: ICMP echo request, id 10022, seq 4, length 64

11:25:33.697970 IP dd.local > 192.168.240.255: ICMP echo request, id 10022, seq 5, length 64

11:25:34.697982 IP dd.local > 192.168.240.255: ICMP echo request, id 10022, seq 6, length 64

此处匹配的是ICMP的广播包，要产生此包，只需要同一个局域网的另一台主机运行ping -b 192.168.240.255即可，当然还可产生组播包，由于没有适合的软件进行模拟产生，在此不举例子。

(4)匹配TCP数据包

TCP同样是TCP/IP协议栈里面最为重要的协议之一，它提供了端到端的可靠数据流，同时很多应用层协议都是把TCP作为底层的通信协议，因为TCP的匹配是非常重要的。

如果想匹配HTTP的通信数据，那只需指定匹配端口为80的条件即可

tcpdump ‘tcp dst port 80’

ylin@ylin:~$ wget http://www.baidu.com 2>1 1 >/dev/null & sudo tcpdump -c 5 ‘tcp port 80’

[1] 10762

tcpdump: verbose output suppressed, use -v or -vv for full protocol decode

listening on eth0, link-type EN10MB (Ethernet), capture size 96 bytes

12:02:47.549056 IP xd-22-43-a8.bta.net.cn.www > ylin.local.47945: S 1202130469:1202130469(0) ack 1132882351 win 2896

12:02:47.549085 IP ylin.local.47945 > xd-22-43-a8.bta.net.cn.www: . ack 1 win 183

12:02:47.549226 IP ylin.local.47945 > xd-22-43-a8.bta.net.cn.www: P 1:102(101) ack 1 win 183

12:02:47.688978 IP xd-22-43-a8.bta.net.cn.www > ylin.local.47945: . ack 102 win 698

12:02:47.693897 IP xd-22-43-a8.bta.net.cn.www > ylin.local.47945: . 1:1409(1408) ack 102 win 724

(5)匹配udp数据包

udp是一种无连接的非可靠的用户数据报，因此udp的主要特征同样是端口，用如下方法可以匹配某一端口

tcpdump ‘upd port 53’ 查看DNS的数据包

ylin@ylin:~$ ping -c 1 www.baidu.com > /dev/null& sudo tcpdump -p udp port 53

[1] 11424

tcpdump: verbose output suppressed, use -v or -vv for full protocol decode

listening on eth0, link-type EN10MB (Ethernet), capture size 96 bytes

12:28:09.221950 IP ylin.local.32853 > 192.168.200.150.domain: 63228+ PTR? 43.22.108.202.in-addr.arpa.

12:28:09.222607 IP ylin.local.32854 > 192.168.200.150.domain: 5114+ PTR? 150.200.168.192.in-addr.arpa. (46)

12:28:09.487017 IP 192.168.200.150.domain > ylin.local.32853: 63228 1/0/0 (80)

12:28:09.487232 IP 192.168.200.150.domain > ylin.local.32854: 5114 NXDomain* 0/1/0 (140)

12:28:14.488054 IP ylin.local.32854 > 192.168.200.150.domain: 60693+ PTR? 69.240.168.192.in-addr.arpa. (45)

12:28:14.755072 IP 192.168.200.150.domain > ylin.local.32854: 60693 NXDomain 0/1/0 (122)

使用ping www.baidu.com目标是产生DNS请求和答应，53是DNS的端口号。

此外还有很多qualitifer是还没有提及的，下面是其它合法的primitive,在tcpdump中是可以直接使用的。

gateway host

匹配使用host作为网关的数据包，即数据报中mac地址（源或目的）为host，但IP报的源和目的地址不是host的数据包。

dst net net

src net net

net net

net net mask netmask

net net/len

匹配IPv4/v6地址为net网络的数据报。

其中net可以为192.168.0.0或192.168这两种形式。如net 192.168 或net 192.168.0.0

net net mask netmask仅对IPv4数据包有效，如net 192.168.0.0 mask 255.255.0.0

net net/len同样只对IPv4数据包有效，如net 192.168.0.0/16

dst portrange port1-port2

src portrange port1-port2

portrange port1-port2

匹配端口在port1-port2范围内的ip/tcp，ip/upd，ip6/tcp和ip6/udp数据包。dst, src分别指明源或目的。没有则表示src or dst

less length 匹配长度少于等于length的报文。

greater length 匹配长度大于等于length的报文。

ip protochain protocol 匹配ip报文中protocol字段值为protocol的报文

ip6 protochain protocol 匹配ipv6报文中protocol字段值为protocol的报文

如tcpdump ‘ip protochain 6 匹配ipv4网络中的TCP报文，与tcpdump ‘ip && tcp’用法一样，这里的&&连接两个primitive。6是TCP协议在IP报文中的编号。

ether broadcast

匹配以太网广播报文

ether multicast

匹配以太网多播报文

ip broadcast

匹配IPv4的广播报文。也即IP地址中主机号为全0或全1的IPv4报文。

ip multicast

匹配IPv4多播报文，也就是IP地址为多播地址的报文。

ip6 multicast

匹配IPv6多播报文，即IP地址为多播地址的报文。

vlan vlan_id

匹配为vlan报文 ，且vlan号为vlan_id的报文

到些为此，我们一直在介绍primitive是如何使用的，也即expression只有一个primitive。通过学会写好每个 primtive，我们就很容易把多个primitive组成一个expression，方法很简单，通过逻辑运算符连接起来就可以了，逻辑运算符有以下 三个：

“&&” 或”and”

“||” 或“or”

“!” 或“not”

并且可通过()进行复杂的连接运算。

如tcpdump ‘ip && tcp’

tcpdump ‘ host 192.168.240.3 &&( tcp port 80 || tcp port 443)’

通过上面的各种primitive，我们可以写出很丰富的条件，如ip, tcp, udp,vlan等等。如IP，可以按址址进行匹，tcp/udp可以按端口匹配。但是，如果我想匹配更细的条件呢？如tcp中只含syn标志，fin标 志的报文呢？上面的primitive恐怕无能为力了。不用怕，tcpdump为你提供最后一个功能最强大的primitive，记住是 primitive，而不是expression。你可以用多个这个的primitive组成更复杂的 expression.

最后一个primitive形式为 expr relop expr

若把这个形式记为A，那么你可这样写tcpdump ‘A1 && A2 && ip src 192.168.200.1’，等等。

下面我们就来分析A这个形式，看看这是如何强大，如果你觉得很乱的话，建议你先用用上面的知识来实际操作几次，要不然就会很乱的，因为expression太复杂了。

形式：expr relop expr

relop表示关系操作符，可以为>, < ,>=,

expr是一个算术表达式，由整数组成和二元运算符（＋，－，＊，/，＆，|, <>)，长度操作，报文数据访问子。同时所有的整数都是无符号的，即0x80000000 和 0xffffffff > 0。为了访问报文中的数据，可使用如下方式：

proto [ expr : size ]

proto表示该问的报文，expr的结果表示该报文的偏移，size为可选的，表示从expr偏移量起的szie个字节，整个表达式为proto报文 中,expr起的szie字节的内容（无符号整数）

下面是expr relop expr这种形式primitive的例子：

‘ether[0] & 1 !=0’ ether报文中第0个bit为1，即以太网广播或组播的primtive。

通过这种方式，我们可以对报文的任何一个字节进行匹配了，因此它的功能是十分强大的。

‘ip[0] = 4’ ip报文中的第一个字节为version，即匹配IPv4的报文，

如果我们想匹配一个syn报文，可以使用：’tcp[13] = 2’，因为tcp的标志位为TCP报文的第13个字节，而syn在这个字节的低1位，故匹配只有syn标志的报文,上述条件是可满要求的，并且比较严格。

如果想匹配ping命令的请求报文，可以使用’icmp[0]=8’，因为icmp报文的第0字符表示类型，当类型值为8时表示为回显示请求。

对于TCP和ICMP中常用的字节，如TCP中的标志位，ICMP中的类型，这个些偏移量有时会忘记。不过tcpdump为你提供更方便的用法，你不用记位这些数字，用字符就可以代替了.

对于ICMP报文，类型字节可以icmptype来表示它的偏称量，上面的primitive可改为’icmp[icmptype] =8’，如果8也记不住怎么办？tcpdump还为该字节的值也提供了字符表示，如’icmp[icmptype] = icmp-echo’。

下面是tcpdump提供的字符偏移量：

icmptype：表示icmp报文中类弄字节的偏移量

icmpcode:表示icmp报文中编码字节的偏移量

tcpflags:表示TCP报文中标志位字节的偏移量

此外，还提供了很多值来对应上面的偏移字节：

ICMP中类型字节的值可以是：

icmp-echoreply, icmp-unreach, icmp-sourcequench, icmp-redi﹔ect, icmp-echo, icmp-routeradvert, icmp-routersolicit,

icmp-timxceed, icmp-paramprob, icmp-tstamp, icmp-tstam﹑reply, icmp-ireq, icmp-ireqreply, icmp-maskreq, icmp-maskreply.

TCP中标志位字节的值可以是：

tcp-fin, tcp-syn, tcp-rst, tcp-push, tcp-ack, tcp-urg.

通过上面的字符表示，我们可以写出下面的primitive

‘tcp[tcpflags] = tcp-syn’ 匹配只有syn标志设置为1的 tcp报文

‘tcp[tcpflags] & (tcp-syn |tcp-ack |tcp-fin) !=0’ 匹配含有syn，或ack或fin标志位的TCP报文

对于IP报文，没有提供字符支持，如果想匹配更细的条件，直接使用数字指字偏移量就可以了，不过要对IP报文有更深入的了解才可以。

学会写primitive后，expression就是小菜一碟了，由一个或多个primitive组成，并且逻辑连接符组成即可：

tcpdump ‘host 192.168.240.91 && icmp[icmptype] = icmp-echo’

tcpdump ‘host 192.168.1.100 && vrrp’

tcpdump ‘ether src 00:00:00:00:00:02 && ether[0] & 1 !=0’

让你随心所欲地使用tcpdump，将不用再从复杂的输出中去挑报文了！

如此，我们可以写出更复杂的表达式来匹配报文，如IP或TCP中的报文id，IP是中的分段标志，ICMP中类型和代码等。

http://www.linuxso.com/command/tcpdump.html