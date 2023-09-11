---
title: IP Filter Based Firewalls HOWTO
author: admin
type: post
date: 2010-11-03T13:15:26+00:00
url: /archives/6537
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - 防火墙

---
Brendan ConoboyErik Fichtner
$FreeBSD: src/share/examples/ipfilter/ipf-howto.txt,v 1.1.2.1 2002/04/27 20:04:18 darrenr Exp $

摘要：本文档向初学者介绍IP Filter防火墙软件，同时介绍一些设计防火墙的基本方法。

**1.1声明**

作者对因该文档所造成的破坏概不负责，该文档只是介绍基于ipfilter的防火墙。如果你觉得你采取的行动不太合适的话，你应该停止阅读该文档，请有资格的安全专家为你安装防火墙。

**1.2 版权**

除非其它情况，该文档版权属于每一个作者。部分或全部文档可以以各种介质复制和重新发布，只要你在所有的拷贝中保存版权声明。任何商业性质的发布应该通知文档的作者。
所有的翻译文档及其它衍生文档应该保留版权声明。

**1.3 哪里可以获得重要文档**

ipf官方网站：
文档的最新版本可以在这找到：

**2. 基本防火墙**

这部分将使您对ipfilter的语法及防火墙原理有个总体的了解。这里讨论的防火墙特点都可以在任何好的防火墙软件中找到。这部分将使您更容易的阅读和理解后面的高级部分。必须提一下的是，这一部分不足以让你建立一个好的防火墙，如果您想建立一个有效率的安全系统您应该阅读高级部分。

**2.1 文件的结构，顺序及优先级**

ipf 有一个配置文件（或者说是一些一次运行的命令）。这个文件沿袭unix风格：每行一条规则，“#”表示注释，多余的空格将被忽略，我们鼓励保持规则文件的可读性。

**2.2 基本的规则处理**

每条规则从上到下顺序执行。这意味着如果你的配置文件是这样的：
block in all
pass in all
计算机将视它视为

block in all
pass in all
就是说，当有一个包进来，第一条规则起作用：

block in all

接下来ipf将包传给下一条规则，第二条规则将起作用：

pass in all

这个时候你也许会问ipf将执行第二条规则吗？如果你熟悉ipfwadm或者ipfw，你也许就不会问这个问题了。也许您将会对包的走向感到困惑，很多包过滤软件只要包符合一条规则，它将不会往下匹配，但是ipf不属于这种类型。
不像其它包过滤器，ipf会为每个包做一个标记，不管这个包通过与否。ipf将遍历整个规则集，除非你打断它的遍历，然后ipf根据最后一条规则决定是通过还是抛弃这个包。当一个包进入一个接口（如rl0，tun0），ipf就开始工作，他先检查这个包然后再检查第一条规则：

block in all

ipf说：“我现在应该阻止这个包”。它又看看第二条规则：

pass in all

“我现在应该通过这个包”。它又看看第三条规则，没有第三条规则，所以这个包根据最后一条规则，向前传递这个包。
现在应该指出的是,即使你的规则是这样的：

block in all
block in all
block in all
block in all
pass in all
这个包依然通过。规则没有累加作用的。最后一条匹配规则总是优先的。

**2.3 控制规则的执行**

如果你有使用其它包过滤器的经验，你也许会发现这种设计令人困惑，会觉得这样匹配速度会有些问题。想象一下如果你有100条规则，而最有用的是前10条，每个包都通过这100条规则是一种很可怕的开销。幸运的是，你可以加一个关键字到任意一条规则里面，只要匹配这条规则，这条规则将马上起作用，而不用跑到最后一条规则。这个关键字是quick。这是一个修改过的规则集，加入了quick：
block in quick all
pass in all
这种情况下，ipf检查第一条规则：

block in quick all

报匹配这条规则并且遍历结束.这个包就被抛弃，没有任何提示、记录。那么下一条规则呢?

pass in all

这条规则绝不会被检查，就像不在配置文件里面。all非常广泛而且quick马上起作用将使后面的规则不起作用。
在这里ipf就像它所配置的那样阻止包的通过，ipf也可以让一些包通过，我们可以稍微改一下规则组来实现。

**2.4 基于ip地址的过滤**

ipf可以在很多方面过滤数据包，我们最熟悉的一种是ip地址，有一些ip地址空间我们是绝对不会与之通信的，(假设)其中一块就是无路由网络，192.168.0.0/16（/16 是子网掩码，ipf也可以使用255.255.0.0）。如果你想阻止192.168.0.0/16，你可以这样：
block in quick from 192.168.0.0/16 to any
pass in all
现在我们有了一个较不严格的规则集，它确实能为我们做些事情。当一个来自1.2.3.4的包进来，查询第一条规则：

block in quick from 192.168.0.0/16 to any

这个包不是来自192.168.\*.\*,所以不匹配。查询第二条规则：

pass in all

这个包匹配，这个包被传到它的目的地。
另一方面，假设我们有一个包来自192.168.1.2，第一条规则被执行：

block in quick from 192.168.0.0/16 to any

匹配，包被丢弃，结束（不执行下一条规则），因为第一条规则包含quick关键字。
现在你可以通过或者阻止相当广泛的地址的数据包，我们在上面的例子中阻止了进入我们防火墙的私有地址的数据包，让我们看看其它的规则：

block in quick from 192.168.0.0/16 to any
block in quick from 172.16.0.0/12 to any
block in quick from 10.0.0.0/8 to any
pass in all
前面的3块ip地址属于私有的ip地址空间。

**2.5 控制接口**

很多公司在它们连入外部网之前就有内部网了。事实上，防火墙的最早应用的地方就是在这里，连接外网和内往之间的机器就是路由器，它与其它机器的差别在于它拥有多个接口。所有你接收到的包以及所有你发出的包都经过网络接口。假设你有3个接口，lo0(loopback),xl0(3com ethernet),tun0(FreeBSD中ppp通常使用的接口),你不需要数据包进入你的tun0接口。
block in quick on tun0 all
pass in all
在这，关键字”on”说明数据是针对tun0接口的。当一个包进入tun0，第一条规则匹配并丢弃这个包，当一个数据包进入lo0或者xl0,第一条规则不匹配，第二条规则匹配，数据报通过。

**2.6 联合ip地址和接口**

一个防火墙匹配的规则标准（如ip，port，接口）越多,防火墙越严谨，也许你需要通过tun0传输数据，但是不想让来自192.168.0.0/16的数据包通过。以下规则是一个防火墙的开始：
block in quick on tun0 from 192.168.0.0/16 to any
pass in all
根据上面的规则，我们用tun0来阻止通过tun0接口的数据包，当一个来自192.168.0.0/16的数据包到达xl0，它将会通过。现在我们可以根据接口建立一个阻止或者允许大范围地址通过的防火墙：

block in quick on tun0 from 192.168.0.0/16 to any
block in quick on tun0 from 172.16.0.0/12 to any
block in quick on tun0 from 10.0.0.0/8 to any
block in quick on tun0 from 127.0.0.0/8 to any
block in quick on tun0 from 0.0.0.0/8 to any
block in quick on tun0 from 169.254.0.0/16 to any
block in quick on tun0 from 192.0.2.0/24 to any
block in quick on tun0 from 204.152.64.0/23 to any
block in quick on tun0 from 224.0.0.0/3 to any
pass in all
你已经了解前3块地址地址空间，第4块是回路地址（浪费了大块的A类网络地址），很多软件通过127.0.0.1与本机进行通信，所以应该阻止源地址是外部地址的数据包。第五块地址0.0.0.0/8，不应该出现在因特网上，大部分的ip堆栈把0.0.0.0/32当作默认网关，其它的0.\*.\*.*，不同的系统有不同的奇怪的处理方法。169.254.0.0/16由IANA分配给那些通过DHCP未能获得ip地址的机器作为自动获得的ip地址。特别是windows操作系统经常使用该ip地址段，当它们是通过DHCP获得ip地址，但是未能找到DHCP服务器。192.0.2.0/24是作者用来举例的ip地址段，同样它也是预留地址。20.20.20.4/24,204.152.64.0/23是sun公司预留的ip地址，是否阻止由你自己决定。224.0.0.0/3主要用于多播地址，几乎包括了D类，E类地址，其它的D类地址可以查找RFC1166。
包过滤有一个非常重要的原则，就是刚才提过的阻止私有地址的通过(除非你确信有跟私有地址进行通信)，当你知道某种类型的数据的源地址，你应该建立只允许来自这个地址的数据通过。例如非路由地址，源地址是10.0.0.0/8的数据包是不应该到达tun0的，因为你没办法对这个数据包进行回复，这是一个非法的数据包，127.0.0.0/8也是这种类型。
很多软件根据数据包的原始地址进行认证，当你有一个内部网20.20.20.0/24，只有内部数据的交换，但是有数据包想要离开内部网，例如有个源地址是20.20.20.0/24想通过ppp拨号，这个时候你有理由抛弃它，这种类型的数据包是不应该到达它的目的地址的，你可以简单的用ipf来实现。规则是这样的：

block in quick on tun0 from 192.168.0.0/16 to any
block in quick on tun0 from 172.16.0.0/12 to any
block in quick on tun0 from 10.0.0.0/8 to any
block in quick on tun0 from 127.0.0.0/8 to any
block in quick on tun0 from 0.0.0.0/8 to any
block in quick on tun0 from 169.254.0.0/16 to any
block in quick on tun0 from 192.0.2.0/24 to any
block in quick on tun0 from 204.152.64.0/23 to any
block in quick on tun0 from 224.0.0.0/3 to any
block in quick on tun0 from 20.20.20.0/24 to any
pass in all

**2.7 双向过滤，关键字“out”**

到现在我们已经通过或者阻止进入防火墙的数据包，必须说明的是，进入的数据包到达的是防火墙的任意一个接口。相反，出去的数据包离开防火墙的任意一个接口(不管是本地产生的还是仅仅是通过防火墙的数据包)，这就是说我们可以过滤进入防火墙的数据包也可以过滤离开防火墙的数据包。
现在我们知道可以像过滤进入的数据包那样过滤离开的数据包，我们可以想到的一个用处就是可以阻止伪装的数据包离开我们自己的网络。如果外部有些机器想通过ipf路由一个目的地址是192.168.0.0/16的数据包，我们干嘛不抛弃它呢，最坏的情况只是浪费一些带宽：
block out quick on tun0 from any to 192.168.0.0/16
block out quick on tun0 from any to 172.16.0.0/12
block out quick on tun0 from any to 10.0.0.0/8
block out quick on tun0 from any to 0.0.0.0/8
block out quick on tun0 from any to 127.0.0.0/8
block out quick on tun0 from any to 169.254.0.0/16
block out quick on tun0 from any to 192.0.2.0/24
block out quick on tun0 from any to 204.152.64.0/23
block out quick on tun0 from any to 224.0.0.0/3
block out quick on tun0 from !20.20.20.0/24 to any
最勉强的观点是这样做并不会提高你的安全性，但是它可以提高其它人的安全性，而且最好是这样做。另外一个观点是因为从你的站点没有人可以发送伪装的数据包，你被crackers攻击的目标就小了。你将会发现很多阻止数据包离开防火墙的用处。有件事你必须记在心上，那就是不管是进来的还是出去的数据包都是相对于你的防火墙的，而不是其它机器。

**2.8 记录，关机字”log”**

到现在所有的数据包都是悄悄的通过或者被阻止，通常你想知道你是否被攻击，但是我并不想记录所有的数据包，而且我想知道被我阻止的来自于20.20.20.0/24数据包的情况。为了达到这个目的，我加入关键字log
block in quick on tun0 from 192.168.0.0/16 to any
block in quick on tun0 from 172.16.0.0/12 to any
block in quick on tun0 from 10.0.0.0/8 to any
block in quick on tun0 from 127.0.0.0/8 to any
block in quick on tun0 from 0.0.0.0/8 to any
block in quick on tun0 from 169.254.0.0/16 to any
block in quick on tun0 from 192.0.2.0/24 to any
block in quick on tun0 from 204.152.64.0/23 to any
block in quick on tun0 from 224.0.0.0/3 to any
block in log quick on tun0 from 20.20.20.0/24 to any
pass in all
现在，我们的防火墙能够很好的阻止来自不明地址的数据包，但是我们还有很多事情要做，我们要做的第一件事情是我们要抛弃来自20.20.20.0/32和20.20.20.255/32的数据包，防止smurf攻击(关于smurf攻击请看相关资料)：

block in quick on tun0 from 192.168.0.0/16 to any
block in quick on tun0 from 172.16.0.0/12 to any
block in quick on tun0 from 10.0.0.0/8 to any
block in quick on tun0 from 127.0.0.0/8 to any
block in quick on tun0 from 0.0.0.0/8 to any
block in quick on tun0 from 169.254.0.0/16 to any
block in quick on tun0 from 192.0.2.0/24 to any
block in quick on tun0 from 204.152.64.0/23 to any
block in quick on tun0 from 224.0.0.0/3 to any
block in log quick on tun0 from 20.20.20.0/24 to any
block in log quick on tun0 from any to 20.20.20.0/32
block in log quick on tun0 from any to 20.20.20.255/32
pass in all
**2.9 基于接口的双向过滤**

如果你想建立一个防火墙规则组，你应该考虑到每个方向每个接口。ipfilter默认的状态是通过所有的数据包，我们不应该依赖于默认规则，应该考虑每一个细节，每一个接口，直到所有的情况都包括进去。
首先我们从lo0开始，这个接口是用于本系统中程序之间的通信，让它自由通过：
pass out quick on lo0
pass in quick on lo0
接着是xl0，在后面我们将对它进行严格的限制，在这里我们假设我们的本地网可以信任,像lo0一样处理：

pass out quick on xl0
pass in quick on xl0
最后是tun0，在上面我们已经对tun0进行限制，合并如下:

block in quick on tun0 from 192.168.0.0/16 to any
block in quick on tun0 from 172.16.0.0/12 to any
block in quick on tun0 from 10.0.0.0/8 to any
block in quick on tun0 from 127.0.0.0/8 to any
block in quick on tun0 from 0.0.0.0/8 to any
block in quick on tun0 from 169.254.0.0/16 to any
block in quick on tun0 from 192.0.2.0/24 to any
block in quick on tun0 from 204.152.64.0/23 to any
block in quick on tun0 from 224.0.0.0/3 to any
block in log quick on tun0 from 20.20.20.0/24 to any
block in log quick on tun0 from any to 20.20.20.0/32
block in log quick on tun0 from any to 20.20.20.255/32
pass in all
这是一个很有效的过滤规则，防止20.20.20.0/24被伪装。下面的例子我们为了简便将只考虑一个方向，当你创建防火墙规则时应该考虑每个方向，每个接口。

**2.10 控制协议；关键字”proto”**

拒绝服务攻击跟缓冲溢出攻击一样猖獗，很多DOS攻击是利用了操作系统TCP/IP栈的失常，它通常是使用icmp包。我们为什么不阻止所有icmp的通过呢？
block in log quick on tun0 proto icmp from any to any

有多少icmp包从tun0进入将被记录并且抛弃。

2.11 用icmp-type关键字对icmp进行过滤

当然抛弃所有的icmp不是一个好主意，因为它在某些方面还是有用的。或许你想抛弃那些没用的icmp类型。如果你想要让ping和traceroute正常工作，你应该让icmp的types 0和11通过。严格来讲，这样做也不太好，权衡安全性和便利性，ipf可以这样做：
pass in quick on tun0 proto icmp from any to 20.20.20.0/24 icmp-type 0
pass in quick on tun0 proto icmp from any to 20.20.20.0/24 icmp-type 11
记得规则的顺序是很重要的。如果我们加入关键字”quick”我们必须在block之前pass,我们需要这样安排规则的顺序

pass in quick on tun0 proto icmp from any to 20.20.20.0/24 icmp-type 0
pass in quick on tun0 proto icmp from any to 20.20.20.0/24 icmp-type 11
block in log quick on tun0 proto icmp from any to any
把这三条规则加入上面的防止欺骗的规则中需要一些机巧。如果我们将这三条规则放在最前面会有些问题

pass in quick on tun0 proto icmp from any to 20.20.20.0/24 icmp-type 0
pass in quick on tun0 proto icmp from any to 20.20.20.0/24 icmp-type 11
block in log quick on tun0 proto icmp from any to any
block in quick on tun0 from 192.168.0.0/16 to any
block in quick on tun0 from 172.16.0.0/12 to any
block in quick on tun0 from 10.0.0.0/8 to any
block in quick on tun0 from 127.0.0.0/8 to any
block in quick on tun0 from 0.0.0.0/8 to any
block in quick on tun0 from 169.254.0.0/16 to any
block in quick on tun0 from 192.0.2.0/24 to any
block in quick on tun0 from 204.152.64.0/23 to any
block in quick on tun0 from 224.0.0.0/3 to any
block in log quick on tun0 from 20.20.20.0/24 to any
block in log quick on tun0 from any to 20.20.20.0/32
block in log quick on tun0 from any to 20.20.20.255/32
pass in all
这个问题是来自192.168.0.0/16的icmp type 0的数据包根据第一条规则将会通过，第四条规则将不会起作用。同样icmp也会通过并到达20.20.20.0/24，这样会向恶意的smurf攻击开了后门，而且最后两条规则也不起作用。为了避免这样的情况发生，我们将icmp规则放到防止欺骗的规则后面:

block in quick on tun0 from 192.168.0.0/16 to any
block in quick on tun0 from 172.16.0.0/12 to any
block in quick on tun0 from 10.0.0.0/8 to any
block in quick on tun0 from 127.0.0.0/8 to any
block in quick on tun0 from 0.0.0.0/8 to any
block in quick on tun0 from 169.254.0.0/16 to any
block in quick on tun0 from 192.0.2.0/24 to any
block in quick on tun0 from 204.152.64.0/23 to any
block in quick on tun0 from 224.0.0.0/3 to any
block in log quick on tun0 from 20.20.20.0/24 to any
block in log quick on tun0 from any to 20.20.20.0/32
block in log quick on tun0 from any to 20.20.20.255/32
pass in quick on tun0 proto icmp from any to 20.20.20.0/24 icmp-type 0
pass in quick on tun0 proto icmp from any to 20.20.20.0/24 icmp-type 11
block in log quick on tun0 proto icmp from any to any
pass in all
因为我们在icmp之前已经阻止了伪装的数据包，一个伪装的数据包将不会到达icmp规则。记住规则的顺序这一点很重要。

**2.12 tcp，udp端口；关键字”port”**

我们已经基于协议对数据包进行过滤，我们还可以基于协议的具体方面进行过滤。最有用的方面是端口号，rsh，rlogin，telnet服务通常是很有用的，但是由于网络嗅探和欺骗攻击，这些服务隐含这一些不安全的因素，一个很重要的妥协方法是只在内部网运行这些服务，阻止外网的访问。这很容易做到，因为rsh，rlogin，telnet使用专用的端口号(514，513，23)。容易建立规则：
block in log quick on tun0 proto tcp from any to 20.20.20.0/24 port = 513
block in log quick on tun0 proto tcp from any to 20.20.20.0/24 port = 514
block in log quick on tun0 proto tcp from any to 20.20.20.0/24 port = 23
这三条规则必须在pass in all之前，这样它们将跟外网隔开(为了简便省略防止欺骗的规则)

pass in quick on tun0 proto icmp from any to 20.20.20.0/24 icmp-type 0
pass in quick on tun0 proto icmp from any to 20.20.20.0/24 icmp-type 11
block in log quick on tun0 proto icmp from any to any
block in log quick on tun0 proto tcp from any to 20.20.20.0/24 port = 513
block in log quick on tun0 proto tcp from any to 20.20.20.0/24 port = 514
block in log quick on tun0 proto tcp from any to 20.20.20.0/24 port = 23
pass in all
你也许想阻止514/udp(syslog)，111/tcp&111/udp(portmap)，515/tcp(lpd)，2049/tcp&udp(nfs)，6000/tcp(X11)等等。你可以获得完整的系统正在侦听端口列表，netstat -a(或者lsof -i，如果你已经安装)。
如果要阻止udp，只要将tcp换成udp，下面是针对syslog的

block in log quick on tun0 proto udp from any to 20.20.20.0/24 port = 514

ipf对于同时处理tcp和udp有一个简单的方法，针对portmap：

block in log quick on tun0 proto tcp/udp from any to 20.20.20.0/24 port = 111

**3. 高级防火墙介绍**

这部分文档跟前面的基础文档联系紧密，不仅包含着高级防火墙的观念，还包含着一些ipfilter特性，一旦你熟悉这部分，你将能够建立一个非常健壮的防火墙。

**3.1 偏执狂：默认拒绝的规则**

根据端口来阻止服务存在一个很大的问题：有时候端口是可以改变的，基于rpc的程序因为端口变化会变得很糟糕的，lockd，statd，甚至是nfsd侦听的端口不是2049。这种情况是很难预料的，更糟糕的是不大可能总是自适应这个变化。我们现在建立一个新的防火墙规则，我们将用的第一条规则是：
block in all

所有的数据报都无法通过。没有什么用处但是很安全。假设你的服务器只跑一个web服务,没有其它的。甚至没有dns查询，只开放80端口，我们可以加入另外一条规则来实现：

block in on tun0 all
pass in quick on tun0 proto tcp from any to 20.20.20.1/32 port = 80
第二条规则将使连接20.20.20.1端口80的数据包通过，其它的数据包则无法通过。作为基本的防火墙，这就够了。

**3.2 状态规则**

防火墙的作用是阻止从A到B的数据包，假设我们有这样一条规则：只要是到达端口23的数据包就可以通过。同样的只要一个数据包有FIN标志就可以通过，我们上面的防火墙并不知道TCP/UDP/ICMP会话的开始段，中间段，结束段。它只是针对所有包的规则。我们希望到达端口23的数据包没有被窃取、改动，就是说我们希望有一个方法将一个正常的TCP/UDP/ICMP数据包跟端口扫描或者是DOS攻击区分开来。这种方法就是所说的keeping state(保持连接状态)。
我们想要做到既方便又安全，很多人已经这么做了，像Ciscos有个established条款，这条款允许已经建立连接的tcp的数据包通过。ipfw也有established,ipfwadm有setup/established(译者注：用于建立连接的标志位/已建立连接的标志位),它们都有这个特点，但是它们的名字容易使人误导。当我们第一次看见的时候，我们会认为我们的包过滤器会追踪一个数据包想要做什么，过滤器会知道一个连接是否已经建立。事实上，它们都是根据数据包的部分例如TCP数据包的标志位来判断一个数据包想要做什么，但是udp/icmp没有用于判断的标志位，无法实现这个功能。如果是这样的话，任何人都可以伪造这些标志位，并使一个数据包通过防火墙。
IPF在这些问题上是怎么处理的呢？ipf不像其它的防火墙，它可以真正的追踪每个数据包并判断一个连接是否已经建立。而且ipf可以处理tcp，udp，icmp，不仅仅是tcp。Ipf称之为keep state(保持状态),规则的关键字就是keep state。
现在你已经知道防火墙检查每一个进入的数据包，同样检查每个离开的数据包。事实上，是状态表检查每个进入或者离开的数据包。状态表是整个防火墙规则中可以通过的TCP/UPD/ICMP会话(连接)表。这会不会是一个严重的安全漏洞呢，事实上这是防火墙中最好的。
所有的tcp/ip会话都有开始，中间及结束(尽管有时候这三个阶段都在同一个包里)，只有结束阶段而没有中间阶段，或者只有中间阶段而没有开始阶段的会话（正常会话）是不可能的。这就是说，你所要做的就是过滤tcp/udp/icmp会话的开始阶段。如果一个会话的开始阶段允许通过，那么它的中间阶段和结束阶段也可以通过(除非ip栈溢出或者机器当机)。Keeping state允许你忽略中间阶段和结束阶段，而仅仅是集中在阻止或者通过一个新的会话。如果一个新的会话通过，那么它所有的后续数据包都被允许通过。这是一个跑ssh服务的例子(仅仅是ssh)：
block out quick on tun0 all
pass in quick on tun0 proto tcp from any to 20.20.20.1/32 port = 22 keep state
你会注意到没有pass out规则。尽管如此，这个规则是完整的。这是keeping state的缘故。一旦一个含有SYN(连接的初始阶段)的数据包到达ssh服务器,会话状态被建立并保存(译者注：这样数据包就可以自由进出)。这是另外一个例子：

block in quick on tun0 all
pass out quick on tun0 proto tcp from 20.20.20.1/32 to any keep state
这种情况下，服务器不跑服务。事实上这不是一台服务器，而是客户机。而且这台客户机不想让未经过认证的包进入ip栈。然而客户机需要访问因特网，同时让回复的包通过。这个简单的规则为每个向外的tcp连接建立状态表。当这个状态表建立以后，这个tcp会话就会畅通无阻，而且没有防火墙规则集的检查(译者注：由状态表检查)。udp和icmp同样适用：

block in quick on tun0 all
pass out quick on tun0 proto tcp from 20.20.20.1/32 to any keep state
pass out quick on tun0 proto udp from 20.20.20.1/32 to any keep state
pass out quick on tun0 proto icmp from 20.20.20.1/32 to any keep state
这样就可以使用ping了，我们已经为tcp，udp，icmp建立连接状态(译者注：严格来讲udp，icmp会话没有建立连接状态，只是防火墙保存了连接状态)。现在我们可以向外连接了，攻击者却无法进入。这是非常方便的，因为我们不必跟踪机器在监听哪些端口，而仅仅是跟踪那些我们想让别人能够连接的端口。
state是非常方便的，但是需要一些机巧。在一些奇怪的令人迷惑的用法上，你有可能“搬石头砸自己的脚”。考虑一下下面的规则：

pass in quick on tun0 proto tcp from any to 20.20.20.1/32 port = 23
pass out quick on tun0 proto tcp from any to any keep state
block in quick all
block out quick all
这看起来像是一个好的防火墙规则，我们允许进入并通过防火墙连接23端口，而且允许所有对外的连接。无疑的，所有连接端口23的数据包都将得到回复，防火墙pass out规则将建立一条状态记录，所有的事情将会很完美，至少我们是这样想的。
不幸的是，60秒的空闲时间以后这条状态记录将会关闭(而不是5天)，这是因为这条状态记录无法看到连接端口23的SYN数据包，它只看到SYN ACK(SYN的回复)。IPF很适合于跟踪从开始到结束的tcp连接，但是不擅长对连接的中间进行跟踪，所以应该像这样重写防火墙规则：

pass in quick on tun0 proto tcp from any to 20.20.20.1/32 port = 23 keep state
pass out quick on tun0 proto tcp from any to any keep state
block in quick all
block out quick all
新加上的那条规则将会对每个syn数据包建立状态记录，而且跟我们期望的一样工作得很好，一旦状态引擎发现了一个连接3次握手已经完成，这个连接就会被表记为4/4模式，这种模式意味着一个长时间数据交换的连接已经建立直到这个连接被拆除(ipfstat -s可以看到连接的模式)。

**3.3 有状态的udp**

udp是无状态协议，因此对它建立可靠的状态连接有很大的难度，尽管如此，ipf在这方面做得相当好。当机器A给B发送一个udp包，源端口是X，目的端口是Y，ipf将允许从机器Y到机器X源端口是Y，目的端口是X的回复包通过。这是一条短期的状态记录，仅仅是60秒。
这是一个例子，用nslookup查询[www.3com.com][1]的ip地址：
$nslookup [www.3com.com][1]

一个DNS包产生：

17:54:25.499852 20.20.20.1.2111 > 198.41.0.5.53: 51979+
这个包从20.20.20.1端口2111到192.41.0.5端口53。一条60秒的状态记录被创建。在60秒的时间内从192.41.0.5端口53到20.20.20.1端口2111的包将通过，就像你所看到的：

17:54:25.501209 198.41.0.5.53 > 20.20.20.1.2111: 51979 q: [www.3com.com][1]

回复包符合状态记录，允许通过，回复包通过之后这条状态记录被关闭，不在允许新的数据包进入，即使这个包来自同一个地方。

**3.4 有状态的icmp**

IPFilter像处理tcp，udp一样处理icmp状态，就像你理解的那样。ICMP主要有两种信息类型：请求和回复。 假设我们写了这样一条规则：
pass out on tun0 proto icmp from any to any icmp-type 8 keep state

允许向外发送回复请求(典型的ping),结果是类型为icmp-type 0的包回复并通过。这条状态记录是一种不完全的0/0(相比4/4)状态，它将在60秒的空闲时间后删除。这样当你为每个外出的icmp包保存状态，你将会得到每个icmp的回复。
然而，大部分icmp消息是由错误的udp(有时是tcp)包产生的，而且在3.4.X的ipfilter中错误的icmp消息(超时或端口不可达)会根据udp(有时是tcp)状态记录而通过防火墙。在旧的ipfilter中，如果你想让traceroute正常工作，你需要这样：

pass out on tun0 proto udp from any to any port 33434><33690 keep state
pass in on tun0 proto icmp from any to any icmp-type timex

而现在你只需要为udp保存状态就可以了：

pass out on tun0 proto udp from any to any port 33434><33690 keep state
block in quick on tun0 all
pass out quick on tun0 proto tcp from 20.20.20.1/32 to any keep state
为了防止第三方icmp消息通过一个活动的连接进入你的防火墙，防火墙不仅对进入的icmp消息进行源地址和目的地址的检查(有时包括端口，如果可用的话)，还对icmp消息中的荷载(产生icmp消息的包的一部分)进行检查。

**3.5 FIN扫描检测**；关键字flags，keep frags

先回顾一下上面的4条规则
pass in quick on tun0 proto tcp from any to 20.20.20.1/32 port = 23 keep state
pass out quick on tun0 proto tcp from any to any keep state
block in quick all
block out quick all
这几条规则还不能令人满意，问题在于不仅允许SYN包到达23端口而且允许老的数据包通过。我们可以用关键字flags改变上面的规则：

pass in quick on tun0 proto tcp from any to 20.20.20.1/32 port = 23 flags S keep state
pass out quick on tun0 proto tcp from any to any flags S keep state
block in quick all
block out quick all
现在只有目的地址是20.20.20.1，目的端口23带有SYN标志的tcp包可以通过。SYN标志只存在于tcp会话的第一个数据包(称为tcp第一次握手)，这也是我们所希望，这样做至少有两个好处：首先不是所有的包都可以进入防火墙并把你的状态表搞的乱七八糟。其次，FIN和XMAS扫描将会行不通，因为它们的标志位不是SYN。现在进入的数据包要么是SYN包要么是已经建立连接的数据包。如果其它的数据包进入，很可能是端口扫描或者伪造的数据包，但是也有可能是一个分片。IPF用关键字keep frags就可以处理分片。加上这个关键字，IPF将跟踪那些分片，允许我们需要的分片通过。我们重写一下规则，允许分片并记录伪装包：

pass in quick on tun0 proto tcp from any to 20.20.20.1/32 port = 23 flags S keep state keep frags
pass out quick on tun0 proto tcp from any to any keep state flags S keep frags
block in log quick all
block out log quick all
这几条规则将起作用，因为所有的包在block之前就已经通过了。但是无法发现SYN扫描，如果你感到不爽你可以记录所有的SYN包。

**3.6 回应被阻止的数据包**

现在被我们阻止的包都被丢弃，或者记录，我们没有发送任何回应包给源主机。有时侯这不是最好的办法，因为如果这样做就等于告诉一个攻击者我们有一个包过滤器。最好是我们能够误导使攻击者相信我们没有包过滤器，并且没有提供服务可供攻击。
当一个服务运行在unix系统上，它通常通过回复包让远程主机知道。在tcp中使通过RST包来回复。当阻止一个tcp通过时，实际上IPF返回一个RST给源主机(用关键字return-rst)。现在我们可以这样做
block return-rst in log proto tcp from any to 20.20.20.0/24 port = 23
block in log quick on tun0
pass in all
return-rst只适用于tcp，我们还想用于udp，icmp及其它协议(下面将介绍)，现在远程主机将得到connection refused而不是connection timed out。
当有人给你的系统的一个udp端口发送数据包，防火墙也可能发送一条错误信息。只要你的规则是这样的：

block in log quick on tun0 proto udp from any to 20.20.20.0/24 port = 111

你可以用加入return-icmp关键字的规则来代替上面的规则来发送回复

block return-icmp(port-unr) in log quick on tun0 proto udp from any to 20.20.20.0/24 port = 111

根据tcp/ip的规范，当给服务器某个端口发送一个数据包，而服务器没有服务进程在监听这个端口时将发出port-unreachable。当然你可以用其它的icmp类型，但是port-unreachable可能是最好的。这也是默认的用来回复的icmp类型
然而当你用return-icmp的时候，你将会发现也不是很安全的，因为icmp包包含了防火墙的ip地址，而不是原始数据包的目的地址。这个问题已经在ipfilter3.3以后的版本中得到解决，一个新的关键字return-icmp-as-dest已经加入。这是新的规则：

block return-icmp-as-dest(port-unr) in log on tun0 proto udp from any to 20.20.20.0/24 port = 111

另外，你要慎用回复包，只有在你很清楚你要对什么数据进行回复的时候才能使用。例如：如果你给局域网的广播地址发送return-icmp，局域网将会在短时间内被淹没。

**3.7 日志**

如果你要使用日志设备dev/ipl，记得要加入关键字log.为了要看到日志信息，你必须运行ipmon(或者其它读取/dev/ipl的软件)。一般是使用ipmon -s向syslog写入信息。以ipfilter3.3为例，你甚至可以通过关键字log level控制syslog记录的行为：
block in log level auth.info quick on tun0 from 20.20.20.0/24 to any
block in log level auth.alert quick on tun0 proto tcp from any to 20.20.20.0/24 port = 21
另外你还可以对记录的信息进行裁剪，比如你对是否有人对你的telnet扫描感兴趣,但是对有人扫描你的telnet端口多少次并不感兴趣，你可以log first关键字来记录第一个包。
log的另外一个用处是跟踪你感兴趣的包，并且记录它的头部字段。Ipfilter使用关键字log body可以记录每个包的前128个字节。你应该限制使用body log，因为它会让你的日志变得冗长。

**3.8 合并所有规则**

现在我们有了一个非常严谨的防火墙了，但是它可以更严谨。先前我们去掉的防止欺骗的规则集实际上是很有用的。建议把它加上：
block in on tun0
block in quick on tun0 from 192.168.0.0/16 to any
block in quick on tun0 from 172.16.0.0/12 to any
block in quick on tun0 from 10.0.0.0/8 to any
block in quick on tun0 from 127.0.0.0/8 to any
block in quick on tun0 from 0.0.0.0/8 to any
block in quick on tun0 from 169.254.0.0/16 to any
block in quick on tun0 from 192.0.2.0/24 to any
block in quick on tun0 from 204.152.64.0/23 to any
block in quick on tun0 from 224.0.0.0/3 to any
block in log quick on tun0 from 20.20.20.0/24 to any
block in log quick on tun0 from any to 20.20.20.0/32
block in log quick on tun0 from any to 20.20.20.255/32
pass out quick on tun0 proto tcp/udp from 20.20.20.1/32 to any keep state
pass out quick on tun0 proto icmp from 20.20.20.1/32 to any keep state
pass in quick on tun0 proto tcp from any to 20.20.20.1/32 port = 80 flags S keep state

**3.9 用规则组优化防火墙**

让我们扩展一下我们的防火墙，使我们的防火墙更有用，作为例子我们将改变一下接口名字xl0,xl1,xl2。
xl0接外部网络20.20.20.0/26
xl1用于代理20.20.20.64/26
xl2连接受防火墙保护的网络20.20.20.128/25
我们先定义整个规则，你应该能清楚地理解它：
block in quick on xl0 from 192.168.0.0/16 to any
block in quick on xl0 from 172.16.0.0/12 to any
block in quick on xl0 from 10.0.0.0/8 to any
block in quick on xl0 from 127.0.0.0/8 to any
block in quick on xl0 from 0.0.0.0/8 to any
block in quick on xl0 from 169.254.0.0/16 to any
block in quick on xl0 from 192.0.2.0/24 to any
block in quick on xl0 from 204.152.64.0/23 to any
block in quick on xl0 from 224.0.0.0/3 to any
block in log quick on xl0 from 20.20.20.0/24 to any
block in log quick on xl0 from any to 20.20.20.0/32
block in log quick on xl0 from any to 20.20.20.63/32
block in log quick on xl0 from any to 20.20.20.64/32
block in log quick on xl0 from any to 20.20.20.127/32
block in log quick on xl0 from any to 20.20.20.128/32
block in log quick on xl0 from any to 20.20.20.255/32
pass out on xl0 all

pass out quick on xl1 proto tcp from any to 20.20.20.64/26 port = 80 flags S keep state
pass out quick on xl1 proto tcp from any to 20.20.20.64/26 port = 21 flags S keep state
pass out quick on xl1 proto tcp from any to 20.20.20.64/26 port = 20 flags S keep state
pass out quick on xl1 proto tcp from any to 20.20.20.65/32 port = 53 flags S keep state
pass out quick on xl1 proto udp from any to 20.20.20.65/32 port = 53 keep state
pass out quick on xl1 proto tcp from any to 20.20.20.66/32 port = 53 flags S keep state
pass out quick on xl1 proto udp from any to 20.20.20.66/32 port = 53 keep state
block out on xl1 all
pass in quick on xl1 proto tcp/udp from 20.20.20.64/26 to any keep state

block out on xl2 all
pass in quick on xl2 proto tcp/udp from 20.20.20.128/25 to any keep state

从这个例子中，我们可以看出我的规则集变得越来越臃肿了。如果我们加入跟多的规则时，情况会变得更严重。影响到xl0与xl2之间通信的性能。如果你建立这样的防火墙，你会浪费大量的带宽和cpu时间，你可以通过建立规则分组来优化防火墙的性能。规则组允许你的规则写成树状结构而不是线形结构。树状结构将原来的规则按某种方式(如接口，ip地址)分成不同的组,而每个组有一条组规则，当一个包进入防火墙的时候先检查组规则，如果不符合则跳过整个规则组，这看起来就像一台机器上有好几个防火墙。
我们先从简单的例子开始：

block out quick on xl1 all head 10
pass out quick proto tcp from any to 20.20.20.64/26 port = 80 flags S keep state group 10
block out on xl2 all
在这个简单的例子中，我们可以看出规则组的作用。当一个数据包的目标接口不是xl1的时候,不匹配规则组10组,并跳过第10组,当数据包的目标接口是xl1时匹配规则组10组其它规则短路(不起作用)。用这种方法我们重写上面的规则提高防火墙的性能。

block in quick on xl0 all head 1
block in quick on xl0 from 192.168.0.0/16 to any group 1
block in quick on xl0 from 172.16.0.0/12 to any group 1
block in quick on xl0 from 10.0.0.0/8 to any group 1
block in quick on xl0 from 127.0.0.0/8 to any group 1
block in quick on xl0 from 0.0.0.0/8 to any group 1
block in quick on xl0 from 169.254.0.0/16 to any group 1
block in quick on xl0 from 192.0.2.0/24 to any group 1
block in quick on xl0 from 204.152.64.0/23 to any group 1
block in quick on xl0 from 224.0.0.0/3 to any group 1
block in log quick on xl0 from 20.20.20.0/24 to any group 1
block in log quick on xl0 from any to 20.20.20.0/32 group 1
block in log quick on xl0 from any to 20.20.20.63/32 group 1
block in log quick on xl0 from any to 20.20.20.64/32 group 1
block in log quick on xl0 from any to 20.20.20.127/32 group 1
block in log quick on xl0 from any to 20.20.20.128/32 group 1
block in log quick on xl0 from any to 20.20.20.255/32 group 1
pass in on xl0 all group 1

pass out on xl0 all

block out quick on xl1 all head 10
pass out quick on xl1 proto tcp from any to 20.20.20.64/26 port = 80 flags S keep state group 10
pass out quick on xl1 proto tcp from any to 20.20.20.64/26 port = 21 flags S keep state group 10
pass out quick on xl1 proto tcp from any to 20.20.20.64/26 port = 20 flags S keep state group 10
pass out quick on xl1 proto tcp from any to 20.20.20.65/32 port = 53 flags S keep state group 10
pass out quick on xl1 proto udp from any to 20.20.20.65/32 port = 53 keep state group 10
pass out quick on xl1 proto tcp from any to 20.20.20.66/32 port = 53 flags S keep state
pass out quick on xl1 proto udp from any to 20.20.20.66/32 port = 53 keep state group 10

pass in quick on xl1 proto tcp/udp from 20.20.20.64/26 to any keep state

block out on xl2 all
pass in quick on xl2 proto tcp/udp from 20.20.20.128/25 to any keep state

现在你可以看到规则组起作用了。当xl2网络的主机不是与xl0网络通信时将绕过规则组10，不受规则组10的检查。在不同的情况下，你可以根据协议或者机器或者网络块进行分组。

**3.10 关键字Fastroute**

尽管我们已经转发一些数据包并阻止其它一些数据包，这样看起来，我们的机器像一台路由器，防火墙同样也减少数据包TTL的值(一跳减一)而且告诉外面我们已经收到了数据包。但是我们可以在一些应用中(比如unix的traceroute)隐藏我们的存在，不减少TTL的值。如果我们想让进入的traceroute正常工作，但是我们不想让它知道防火墙的存在，我们可以这样做：
block in quick on xl0 fastroute proto udp from any to any port 33434 >< 33465

关键字fastroute将告诉ipfilter不要让这个包进入unix的ip栈，因为进入unix的ip栈会减少TTL的值。这个数据包会被防火墙偷偷的放到正确的出口，而且不会减少TTL的值。当然ipfilter会根据系统路由表指出数据包从哪个接口出去。
在这个例子中，我们使用了block quick是有理由的。如果我们使用了pass，而且我们在内核中打开ip转发，我们将会有两条这个数据包的出口路径，内核就很有可能”不知所措”。
需要注意的是，大部分unix的内核路由的代码比ipfilter的效率更高，因此这个关键字不是用来提高防火墙的性能，而仅仅是用来隐藏我们自己。

**4. 网络地址转换和代理**

防火墙一个最大的应用是使几台机器可以通过一个公用的外部接口连到外部网络。对于那些熟悉linux的用户来说，这个概念就是ip伪装，对于其他人来说它有一个更模糊的说法”网络地址转换”,简称NAT.
其实ipfilter可以称为NPAT，因为ipfilter不仅对地址进行转换还对端口进行转换，而NAT只是改变了地址。

**4.1 多个地址转换为一个地址**

基本的NAT可以完成Linux ip伪装的功能，可以这样做：
map tun0 192.168.1.0/24 -> 20.20.20.1/32

很简单，只要源地址符合192.168.1.0/24通过tun0出去的数据包，它的源地址都会被改写为20.20.20.1而目的地址不会改变。系统为地址转换维护一个表，这样回复的包都可以根据这个表转换为正确的地址(20.20.20.1转换成内网地址)。
我们刚才写的那条规则有个缺点：在大部分情况下，我们不知道我们外网的ip地址(动态地址)，幸好NAT解决了这个问题，它可以用0/32来代替，当它发现地址是0/32时它就知道应该查找接口的真正地址。

map tun0 192.168.1.0/24 -> 0/32

现在我们可以放心的加载NAT规则了，并且不用做任何修改就可以连到外部网了。你所需要做的仅仅是当你的IP地址改变后运行一下ipf -y
你或许会想到当地址转换的时候，端口有什么变化。以我们现在的规则源端口是不会改变的，在有些场合我们希望端口也能够改变，例如，当你的防火墙上面还有一个防火墙，而我们又需要通过这个防火墙，或者是有很多主机用到了相同的源端口，这个时候就会发生冲突，ipnat可以用关键字portmap解决这个问题：

map tun0 192.168.1.0/24 -> 0/32 portmap tcp/udp 20000:30000

它的作法是将所有的链接都塞入20000到30000之间的端口。

**4.2 多地址影射到地址池**

NAT另外一个用法是将很多的静态地址映射(地址转换)到较少量的地址空间,利用你所学的知识就能够做到:
map tun0 192.168.0.0/16 -> 20.20.20.0/24 portmap tcp/udp 20000:60000

当然有些远程运用需要多个链接来自同一个ip地址(比如192.168.0.31需要访问218.9.121.110，它的并发链接经过NAT之后很有可能来自不同的IP地址)，我们可以告诉NAT静态的映射每个链接，用关键字map-block实现：

map-block tun0 192.168.1.0/24 -> 20.20.20.0/24

**4.3 点对点映射**

bimap tun0 192.168.1.1/32 -> 20.20.20.1/32
**4.4 伪装**

假设我们有一台web服务器地址是20.20.20.5，我们怀疑我们网络的安全性有问题，我们不想在端口80上提供服务，因为它需要用root来运行一小段时间。我们如何让web服务器运行于8000端口呢？客户机如何访问呢？我们可以使用NAT的重定向来解决这个问题，指示所有对20.20.20.5:80访问都指向20.20.20.5:8000，用rdr关键字来实现：
rdr tun0 20.20.20.5/32 port 80 -> 20.20.20.5 port 8000

当然这里我们也可以指定协议，如果我们想重定向一个udp服务而不是tcp(tcp是默认的)。例如，如果我们在防火墙上有一个模仿windows后门的”蜜罐”，我们可以把整个网络都定向到这个地方：

rdr tun0 20.20.20.0/24 port 31337 -> 127.0.0.1 port 31337 udp

rdr有一个相当重要的地方：你不能简单地把重定向用成“反射镜”。例如:
rdr tun0 20.20.20.5/32 port 80 -> 20.20.20.6 port 80 tcp
将不会正常工作，因为.5和.6在同一个局域网段上。首先，一个到达20.20.20.5接口tun0的数据包将会被重定向到20.20.20.6，也就是它的目的地址被修改了，然后被送到ipf过滤规则进行过滤（注意作用顺序，先地址转换后过滤），如果它通过了ipf，那么它就被送到unix路由代码处，这个时候这个数据包目的地址被改了，但是它的目的接口还是tun0，系统就不知道该怎么办了。因此”反射镜”是不能正常工作的。记住：使用rdr时，目的地址必须是从不同的接口离开防火墙。

**4.5 透明代理**

当你在建立一个防火墙的时候，你会认为应该谨慎的使用代理，你可以”绷紧”你的防火墙规则来保护你的内部网。或者你认为你的NAT没有正常的工作，你可以使用重定向：
rdr xl0 0.0.0.0/0 port 21 -> 127.0.0.1 port 21

这条规则是说任何连接ftp的数据包都被重定向到了127.0.0.1
针对ftp的代理有些复杂，因为web浏览器或者其它自动登录类型的客户端不知道如何跟代理通信。有个补丁是针对TIS防火墙的sftp-gw，NAT配合这个补丁能够解决这个问题。很多代理软件是透明代理(如squid)。
当你想强迫你的用户先向代理请求验证时，关键字rdr经常是很有用的。(例如你想让你的工程师能够上网冲浪，但是不希望呼叫中心的员工上网)

**4.6 应用代理**

ftp有两种工作方式，如果想让防火墙后面的ftp正常工作，应该使用应用代理。我们可以让我们的防火墙注意通过它的每一个包，当它发现它正在处理的是主动ftp连接，它能够产生几条临时规则，就像keep state，使得ftp的数据传输能够正常工作。我们需要些这样的规则：
map tun0 192.168.1.0/24 -> 20.20.20.1/32 proxy port ftp ftp/tcp

记得把这条规则写在其它映射规则的前面，否则其它映射规则在这条规则就起作用了，记住ipnat跟ipfilter不同，ipnat是先匹配规则(只要一匹配其它规则就跳过去)。另外rcmd和raudio代理也必须写在其它规则的前面。

**5. 操作过滤规则，ipf**

ipf用来加载ipfilter规则。规则文件可以放在系统的任何地方，但是一般都放在/etc/ipf.rules,/usr/local/etc/ipf.rules,/etc/opt/ipf/ipf.rules
IPfilter可以有两套规则，活动规则和不活动规则。默认情况下所有的操作都是基于活动规则。你可以用ipf -I来使用不活动规则。这两套规则可以用参数-s进行转换。这是非常有用的，你在测试新规则的时候就不用清除老规则。
ipf加参数-r可以删除列表中的规则，但是比较安全的方法是用参数-F清除规则，然后再加载修改后的规则。
加载规则最简单的方法是ipf -Fa -f /etc/ipf.rules.想获得其它操作规则的方法请参考ipf的man page.

**6．加载NAT规则，ipnat**

ipnat用来加载NAT规则。规则文件可以放在系统的任何地方，但是一般都放在/etc/ipnat.rules/usr/local/etc/ipnat.rules，/etc/opt/ipf/ipnat.rules用ipnat加载，也可以用参数-r删除规则。但一般也是清空(参数-C)然后加载，对于活动的映射-C无效，可以用-F来清除。
NAT规则和活动的映射可以用ipnat -l查看。最简单的家在NAT规则的方法是ipnat -CF -f /etc/ipnat.rules.

**7. 监视和调试**

你也许很想知道防火墙到底在干什么，而且如果ipfilter没有状态监视工具的话，它就不是一个完整的防火墙。

**7.1 ipfstat工具**

ipfstat最简的用法是显示一个关于防火墙执行情况的数据表，比如有多少个包通过或则抛弃，它们是否被记录，以及由多少状态条等等。你将看到的是这样一些数据：
\# ipfstat
input packets: blocked 99286 passed 1255609 nomatch 14686 counted 0
output packets: blocked 4200 passed 1284345 nomatch 14687 counted 0

input packets logged: blocked 99286 passed 0
output packets logged: blocked 0 passed 0
packets logged: input 0 output 0
log failures: input 3898 output 0
fragment state(in): kept 0 lost 0
fragment state(out): kept 0 lost 0
packet state(in): kept 169364 lost 0
packet state(out): kept 431395 lost 0
ICMP replies: 0 TCP RSTs sent: 0
Result cache hits(in): 1215208 (out): 1098963
IN Pullups succeeded: 2 failed: 0
OUT Pullups succeeded: 0 failed: 0
Fastroute successes: 0 failures: 0
TCP cksum fails(in): 0 (out): 0
Packet log flags set: (0)
none

ipfstat当然也能够显示你目前的规则列表。参数-i或者-o显示有哪些in规则或者out规则，加上参数-h能够显示更详细的信息，包括每条规则有多少个数据包命中。例如：

\# ipfstat -ho
2451423 pass out on xl0 from any to any
354727 block out on ppp0 from any to any
430918 pass out quick on ppp0 proto tcp/udp from 20.20.20.0/24 to any keep state keep frags
从这里我们可以看出哪些地方可能不正常，ipfstat不能告诉你哪些规则正确或者错误，它只是告诉你由于你的规则而发生了什么事情。为了进一步调试规则，可以使用参数-n，这个参数显示规则的顺序

\# ipfstat -on
@1 pass out on xl0 from any to any
@2 block out on ppp0 from any to any
@3 pass out quick on ppp0 proto tcp/udp from 20.20.20.0/24 to any keep state keep frags
ipfstat的最后一个用处是显示一些关于状态记录的数据。这个参数是-s

\# ipfstat -s
281458 TCP
319349 UDP
0 ICMP
19780145 hits
5723648 misses
0 maximum
0 no memory
1 active
319349 expired
281419 closed
100.100.100.1 -> 20.20.20.1 ttl 864000 pass 20490 pr 6 state 4/4
pkts 196 bytes 17394 987 -> 22 585538471:2213225493 16592:16500
pass in log quick keep state
pkt\_flags & b = 2, pkt\_options & ffffffff = 0
pkt\_security & ffff = 0, pkt\_auth & ffff = 0
从这我们可以看到有一条tcp连接状态。不同的版本输出的内容会略有不同，但是基本的信息是一样的。这条连接的状态是4/4，而其它的状态是不完整的，我们将在后面详细介绍。4/4状态的超时时间是240小时，相当长的一段时间，但它默认是已建立的tcp连接的超时时间。当这条状态空闲时TTL的值每秒减1，最终超时并被删除。当一个连接状态重新启用时，它的TTL值又恢复到864000，必须确保一个活动的连接不会超时。我们还可以看到有196个17K的数据包通过。还有两端的端口号，这个例子是987和22。这意味着这条状态表示一个从100.100.100.1端口987到20.20.20.1端口22的连接。第二行最大的数字是TCP顺序号，保证没有人能够轻易的在这个连接中注入伪装的数据包。TCP的窗口也显示出来

**7.2 ipmon**

ipfstat对收集在系统上发生的事情很有用，但是它还不能够方便且及时的查看日志。ipmon是一个工具，它有能力查看包过滤的日志(关键字log产生的日志)，状态日志，或者是nat日志以及由它们三者共同产生的日志。这个工具可以在前台运行也可以在后台运行(将日志传给syslogd或者一个文件)。如果我们想看状态表的当前情况，可以运行ipmon -o S
\# ipmon -o S
01/08/1999 15:58:57.836053 STATE:NEW 100.100.100.1,53 -> 20.20.20.15,53 PR udp
01/08/1999 15:58:58.030815 STATE:NEW 20.20.20.15,123 -> 128.167.1.69,123 PR udp
01/08/1999 15:59:18.032174 STATE:NEW 20.20.20.15,123 -> 128.173.14.71,123 PR udp
01/08/1999 15:59:24.570107 STATE:EXPIRE 100.100.100.1,53 -> 20.20.20.15,53 PR udp Pkts 4 Bytes 356
01/08/1999 16:03:51.754867 STATE:NEW 20.20.20.13,1019 -> 100.100.100.10,22 PR tcp
01/08/1999 16:04:03.070127 STATE:EXPIRE 20.20.20.13,1019 -> 100.100.100.10,22 PR tcp Pkts 63 Bytes 4604
我们可以看到有一条外部机器向我们的dnssever发送dns请求的状态条，两条xntp ping到时间服务器，一条短暂的向外的ssh连接。
ipmon也可以显示哪些数据包被日志记录了。例如

\# ipmon -o I
15:57:33.803147 ppp0 @0:2 b 100.100.100.103,443 -> 20.20.20.10,4923 PR tcp len 20 1488 -A
它们的含义分别是时间戳 接口 规则 阻止 源地址，端口 -> 目的地址，端口 协议 tcp 包长度 20 1488 ACK
最后我们看一下NAT表

\# ipmon -o N
01/08/1999 05:30:02.466114 @2 NAT:RDR 20.20.20.253,113 <- -> 20.20.20.253,113 [100.100.100.13,45816]
01/08/1999 05:30:31.990037 @2 NAT:EXPIRE 20.20.20.253,113 <- -> 20.20.20.253,113 [100.100.100.13,45816] Pkts 10 Bytes 455
这是一条重定向的规则，用于在我们的防火墙后面提供规则的场合。

**8．ipfilter的特殊用法**

**8.1 基于服务器和标志的deep state**

保存状态很有用，但是很容易犯错。通常，你在第一条与数据包进行交互的规则中加入keep state，一个普遍的错误是混合了状态跟踪和基于标志的过滤：
block in all
pass in quick proto tcp from any to 20.20.20.20/32 port = 23 flags S
pass out all keep state
这几条规则的本意是允许客户机与服务器20.20.20.20进行telnet连接，如果你在试验这规则你会发现它能够马上起作用，由于我们是对SYN进行匹配的，状态条是不能够做到4/4的(不完整状态)，这个状态默认的超时是60秒。
我们可以重写规则来解决这个问题：
1)
block in all
pass in quick proto tcp from any to 20.20.20.20/32 port = 23 keep state
block out all
或者
2)
block in all
pass in quick proto tcp from any to 20.20.20.20/32 port = 23 flags S keep state
pass out all keep state
这两组规则的任何一组都可以为每个连接建立完整的状态条。

**8.2 解决ftp**

ftp有两种不同的传输模式，防火墙管理员不得不解决ftp的这些问题。更糟的是ftp客户机和服务器的问题是不一样的。
ftp协议里面有两种数据传输模式，主动和被动。主动传输是服务器向客户机的一个开放端口进行连接然后传数据。相反地，被动传输是客户机向服务器发起连接并接受数据。

**8.2.1 运行ftp服务**

在运行ftp服务中，处理主动ftp的连接是比较简单的。而处理被动ftp则是一个大问题。首先我们先讨论怎样处理主动ftp，然后是被动ftp。通常主动ftp就像一个进入的http或者smtp连接：我们只要打开ftp端口并keep state就可以了：
pass in quick proto tcp from any to 20.20.20.20/32 port = 21 flags S keep state
pass out proto tcp all keep state
这两条规则允许主动ftp连接，这是最常见的类型。
下面是处理被动ftp连接。web浏览器默认是这种情况，它变得越来越流行，因此我们必须支持它。被动ftp的问题是针对每个被动连接，ftp服务器必须重开一个新端口(通常是1023以上)。这就像是在服务器上提供一个新的服务。如果我们的防火墙默认策略是禁止的，那么这个新的服务将被阻止，ftp会话就中断了。但是我们还有别的办法可以解决这个问题。
也许有人会打开所有1023以上的端口来解决这个问题。事实上，这个方法确实能够解决问题，尽管它令人不太满意：

pass in quick proto tcp from any to 20.20.20.20/32 port > 1023 flags S keep state
pass out proto tcp all keep state
打开所有1023以上的端口，我们将面临一些潜在的危险。端口1-1023是提供给服务进程使用的，众多的程序决定使用1023以上的端口，比如nfsd和X
幸运的是ftp服务进程可以自己决定哪个端口分配给被动连接。这意味着你可以仅仅打开15001-19999端口作为ftp的端口，而不是打开所有1023以上的端口。在wu-ftpd里面，可以在ftpaccess中指定被动端口。请参考ftpaccess的手册。ipfilter所需要做的仅仅是建立相应的规则：

pass in quick proto tcp from any to 20.20.20.20/32 port 15000 >< 20000 flags S keep state
pass out proto tcp all keep state
如果这样还不能使你满意的话，你只好修改ipf来支持ftp或者修改ftp来支持ipf。

**8.2.2 运行ftp客户程序**

尽管ipf对ftp服务的支持不甚完美，但是从3.3.3以后的版本对ftp客户软件的支持已经比较完善。就像ftp服务一样，ftp客户应用也有两个传输模式：主动和被动。
从防火墙的观点来看，ftp最简单的传输模式是被动模式。假设你对所有外出的tcp连接keep state，被动传输就可以正常工作了。如果你还没有这样做的话可以使用下面的规则：
pass out proto tcp all keep state

主动模式有一点麻烦，主动模式使服务器打开第二个连接与客户机进行数据交换。当它们之间有防火墙时就会出现问题。为了解决这个问题，ipfilter包含了一个ipnat代理，这个代理临时在防火墙上打开一个洞，使得ftp服务器可以顺利的连接客户机。甚至是你没有使用ipnat来做地址转换，这个代理也是有效的。下面这条规则加入ipnat的配置文件当中(ep0是外网接口):

map ep0 0/0 -> 0/32 proxy port 21 ftp/tcp

关于ipfilter内部代理的细节，请看3.6

**8.3 内核参数**

有一些内核参数是建立ipf所必需的，还有一些让我们可以方便的了解建立防火墙的信息。其中主要的一项是打开ip转发，否则ipf几乎什么也做不了，因为ip栈不会真的路由数据包。
ip转发：
openbsd:
net.inet.ip.forwarding=1

freebsd:
net.inet.ip.forwarding=1

netbsd:
net.inet.ip.forwarding=1

solaris:
ndd -set /dev/ip ip_forwarding 1

端口调整：

openbsd:
net.inet.ip.portfirst = 25000

freebsd:
net.inet.ip.portrange.first = 25000 net.inet.ip.portrange.last = 49151

netbsd:
net.inet.ip.anonportmin = 25000 net.inet.ip.anonportmax = 49151

solaris:
ndd -set /dev/tcp tcp\_smallest\_anon_port 25000
ndd -set /dev/tcp tcp\_largest\_anon_port 65535

其它有用的参数：

openbsd:
net.inet.ip.sourceroute = 0
net.inet.ip.directed-broadcast = 0

openbsd:
net.inet.ip.sourceroute = 0
net.inet.ip.directed-broadcast = 0

freebsd:
net.inet.ip.sourceroute=0
net.ip.accept_sourceroute=0

netbsd:
net.inet.ip.allowsrcrt=0
net.inet.ip.forwsrcrt=0
net.inet.ip.directed-broadcast=0
net.inet.ip.redirect=0

solaris:
ndd -set /dev/ip ip\_forward\_directed_broadcasts 0
ndd -set /dev/ip ip\_forward\_src_routed 0
ndd -set /dev/ip ip\_respond\_to\_echo\_broadcast 0

**另外，freebsd有一些sysctl变量：**

net.inet.ipf.fr_flags: 0
net.inet.ipf.fr_pass: 514
net.inet.ipf.fr_active: 0
net.inet.ipf.fr_tcpidletimeout: 864000
net.inet.ipf.fr_tcpclosewait: 60
net.inet.ipf.fr_tcplastack: 20
net.inet.ipf.fr_tcptimeout: 120
net.inet.ipf.fr_tcpclosed: 1
net.inet.ipf.fr_udptimeout: 120
net.inet.ipf.fr_icmptimeout: 120
net.inet.ipf.fr_defnatage: 1200
net.inet.ipf.fr_ipfrttl: 120
net.inet.ipf.ipl_unreach: 13net.inet.ipf

 [1]: http://www.3com.com/