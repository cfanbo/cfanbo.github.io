---
title: Linux中Iptables命令详解
author: admin
type: post
date: 2010-08-22T09:51:01+00:00
url: /archives/5243
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - iptables

---
手册：

用iptables -ADC 来指定链的规则，-A添加 -D删除 -C 修改

iptables – [RI] chain rule num rule-specification[option]
用iptables – RI 通过规则的顺序指定

iptables -D chain rule num[option]
删除指定规则

iptables -\[LFZ\] \[chain\][option]
用iptables -LFZ 链名 [选项]

iptables -[NX] chain
用 -NX 指定链

iptables -P chain target[options]
指定链的默认目标

iptables -E old-chain-name new-chain-name
-E 旧的链名 新的链名

用新的链名取代旧的链名

**说明**

Iptalbes 是用来设置、维护和检查Linux内核的IP包过滤规则的。

可以定义不同的表，每个表都包含几个内部的链，也能包含用户定义的链。每个链都是一个规则列表，对对应的包进行匹配：每条规则指定应当如何处理与之相匹配的包。这被称作’target’（目标），也可以跳向同一个表内的用户定义的链。

**TARGETS**

防火墙的规则指定所检查包的特征，和目标。如果包不匹配，将送往该链中下一条规则检查；如果匹配,那么下一条规则由目标值确定.该目标值可以是用户定义的链名,或是某个专用值,如ACCEPT[通过], DROP[删除], QUEUE[排队], 或者 RETURN[返回]。

ACCEPT 表示让这个包通过。DROP表示将这个包丢弃。QUEUE表示把这个包传递到用户空间。RETURN表示停止这条链的匹配，到前一个链的规则重新开始。如果到达了一个内建的链(的末端)，或者遇到内建链的规则是RETURN，包的命运将由链准则指定的目标决定。

**TABLES**

当前有三个表（哪个表是当前表取决于内核配置选项和当前模块)。

-t table

这个选项指定命令要操作的匹配包的表。如果内核被配置为自动加载模块，这时若模块没有加载，(系统)将尝试(为该表)加载适合的模块。这些表如下：filter,这是默认的表，包含了内建的链INPUT（处理进入的包）、FORWORD（处理通过的包）和OUTPUT（处理本地生成的包）。nat,这个表被查询时表示遇到了产生新的连接的包,由三个内建的链构成：PREROUTING (修改到来的包)、OUTPUT（修改路由之前本地的包）、POSTROUTING（修改准备出去的包）。mangle 这个表用来对指定的包进行修改。它有两个内建规则：PREROUTING（修改路由之前进入的包）和OUTPUT（修改路由之前本地的包）。

**OPTIONS**

这些可被iptables识别的选项可以区分不同的种类。

**COMMANDS**

这些选项指定执行明确的动作：若指令行下没有其他规定,该行只能指定一个选项.对于长格式的命令和选项名,所用字母长度只要保证iptables能从其他选项中区分出该指令就行了。

-A -append

在所选择的链末添加一条或更多规则。当源（地址）或者/与 目的（地址）转换为多个地址时，这条规则会加到所有可能的地址(组合)后面。

-D -delete

从所选链中删除一条或更多规则。这条命令可以有两种方法：可以把被删除规则指定为链中的序号(第一条序号为1),或者指定为要匹配的规则。

-R -replace

从选中的链中取代一条规则。如果源（地址）或者/与 目的（地址）被转换为多地址，该命令会失败。规则序号从1开始。

-I -insert

根据给出的规则序号向所选链中插入一条或更多规则。所以，如果规则序号为1，规则会被插入链的头部。这也是不指定规则序号时的默认方式。

-L -list

显示所选链的所有规则。如果没有选择链，所有链将被显示。也可以和z选项一起使用，这时链会被自动列出和归零。精确输出受其它所给参数影响。

-F -flush

清空所选链。这等于把所有规则一个个的删除。

–Z -zero

把所有链的包及字节的计数器清空。它可以和 -L配合使用，在清空前察看计数器，请参见前文。

-N -new-chain

根据给出的名称建立一个新的用户定义链。这必须保证没有同名的链存在。

-X -delete-chain

删除指定的用户自定义链。这个链必须没有被引用，如果被引用，在删除之前你必须删除或者替换与之有关的规则。如果没有给出参数，这条命令将试着删除每个非内建的链。

-P -policy

设置链的目标规则。

-E -rename-chain

根据用户给出的名字对指定链进行重命名，这仅仅是修饰，对整个表的结构没有影响。TARGETS参数给出一个合法的目标。只有非用户自定义链可以使用规则，而且内建链和用户自定义链都不能是规则的目标。

-h Help.

帮助。给出当前命令语法非常简短的说明。

**PARAMETERS[****参数]**

以下参数构成规则详述，如用于add、delete、replace、append 和 check命令。

-p -protocal [!]protocol

规则或者包检查(待检查包)的协议。指定协议可以是tcp、udp、icmp中的一个或者全部，也可以是数值，代表这些协议中的某一个。当然也可以使用在/etc/protocols中定义的协议名。在协议名前加上”!”表示相反的规则。数字0相当于所有all。Protocol all会匹配所有协议，而且这是缺省时的选项。在和check命令结合时，all可以不被使用。

-s -source [!] address[/mask]

指定源地址，可以是主机名、网络名和清楚的IP地址。mask说明可以是网络掩码或清楚的数字，在网络掩码的左边指定网络掩码左边”1″的个数，因此，mask值为24等于255.255.255.0。在指定地址前加上”!”说明指定了相反的地址段。标志 –src 是这个选项的简写。

-d –destination [!] address[/mask]

指定目标地址，要获取详细说明请参见 -s标志的说明。标志 –dst 是这个选项的简写。

-j –jump target

-j 目标跳转

指定规则的目标；也就是说，如果包匹配应当做什么。目标可以是用户自定义链（不是这条规则所在的），某个会立即决定包的命运的专用内建目标，或者一个扩展（参见下面的EXTENSIONS）。如果规则的这个选项被忽略，那么匹配的过程不会对包产生影响，不过规则的计数器会增加。

-i -in-interface [!] [name]

i -进入的（网络）接口 \[!\]\[名称\]

这是包经由该接口接收的可选的入口名称，包通过该接口接收（在链INPUT、FORWORD和PREROUTING中进入的包）。当在接口名前使用”!”说明后，指的是相反的名称。如果接口名后面加上”+”，则所有以此接口名开头的接口都会被匹配。如果这个选项被忽略，会假设为”+”，那么将匹配任意接口。

-o –out-interface [!][name]

-o –输出接口[名称]

这是包经由该接口送出的可选的出口名称，包通过该口输出（在链FORWARD、OUTPUT和POSTROUTING中送出的包）。当在接口名前使用”!”说明后，指的是相反的名称。如果接口名后面加上”+”，则所有以此接口名开头的接口都会被匹配。如果这个选项被忽略，会假设为”+”，那么将匹配所有任意接口。

[!] -f, –fragment

[!] -f –分片

这意味着在分片的包中，规则只询问第二及以后的片。自那以后由于无法判断这种把包的源端口或目标端口（或者是ICMP类型的），这类包将不能匹配任何指定对他们进行匹配的规则。如果”!”说明用在了”-f”标志之前，表示相反的意思。

**OTHER OPTIONS**

**其他选项**

还可以指定下列附加选项：

-v –verbose

-v –详细

详细输出。这个选项让list命令显示接口地址、规则选项（如果有）和TOS（Type of Service）掩码。包和字节计数器也将被显示，分别用K、M、G(前缀)表示1000、1,000,000和1,000,000,000倍（不过请参看-x标志改变它），对于添加,插入,删除和替换命令，这会使一个或多个规则的相关详细信息被打印。

-n –numeric

-n –数字

数字输出。IP地址和端口会以数字的形式打印。默认情况下，程序试显示主机名、网络名或者服务（只要可用）。

-x -exact

-x -精确

扩展数字。显示包和字节计数器的精确值，代替用K,M,G表示的约数。这个选项仅能用于 -L 命令。

–line-numbers

当列表显示规则时，在每个规则的前面加上行号，与该规则在链中的位置相对应。

**MATCH EXTENSIONS[对应的扩展]**

iptables能够使用一些与模块匹配的扩展包。以下就是含于基本包内的扩展包，而且他们大多数都可以通过在前面加上!来表示相反的意思。

**tcp**

当 –protocol tcp 被指定,且其他匹配的扩展未被指定时,这些扩展被装载。它提供以下选项：

–source-port [!] [port[:port]]

源端口或端口范围指定。这可以是服务名或端口号。使用格式端口：端口也可以指定包含的（端口）范围。如果首端口号被忽略，默认是”0″，如果末端口号被忽略，默认是”65535″，如果第二个端口号大于第一个，那么它们会被交换。这个选项可以使用 –sport的别名。

–destionation-port [!] [port:[port]]

目标端口或端口范围指定。这个选项可以使用 –dport别名来代替。

–tcp-flags [!] mask comp

匹配指定的TCP标记。第一个参数是我们要检查的标记，一个用逗号分开的列表，第二个参数是用逗号分开的标记表,是必须被设置的。标记如下：SYN ACK FIN RST URG PSH ALL NONE。因此这条命令：iptables -A FORWARD -p tcp –tcp-flags SYN, ACK, FIN, RST SYN只匹配那些SYN标记被设置而ACK、FIN和RST标记没有设置的包。

[!] –syn

只匹配那些设置了SYN位而清除了ACK和FIN位的TCP包。这些包用于TCP连接初始化时发出请求；例如，大量的这种包进入一个接口发生堵塞时会阻止进入的TCP连接，而出去的TCP连接不会受到影响。这等于 –tcp-flags SYN, RST, ACK SYN。如果”–syn”前面有”!”标记，表示相反的意思。

–tcp-option [!] number

匹配设置了TCP选项的。

**udp**

当protocol udp 被指定,且其他匹配的扩展未被指定时,这些扩展被装载,它提供以下选项：

–source-port [!] [port:[port]]

源端口或端口范围指定。详见 TCP扩展的–source-port选项说明。

–destination-port [!] [port:[port]]

目标端口或端口范围指定。详见 TCP扩展的–destination-port选项说明。

icmp

当protocol icmp被指定,且其他匹配的扩展未被指定时,该扩展被装载。它提供以下选项：

–icmp-type [!] typename

这个选项允许指定ICMP类型，可以是一个数值型的ICMP类型，或者是某个由命令iptables -p icmp -h所显示的ICMP类型名。

mac

–mac-source [!] address

匹配物理地址。必须是XX:XX:XX:XX:XX这样的格式。注意它只对来自以太设备并进入PREROUTING、FORWORD和INPUT链的包有效。

limit

这个模块匹配标志用一个标记桶过滤器一一定速度进行匹配,它和LOG目标结合使用来给出有限的登陆数.当达到这个极限值时,使用这个扩展包的规则将进行匹配.(除非使用了”!”标记)

–limit rate

最大平均匹配速率：可赋的值有’/second’, ‘/minute’, ‘/hour’, or ‘/day’这样的单位，默认是3/hour。

–limit-burst number

待匹配包初始个数的最大值:若前面指定的极限还没达到这个数值,则概数字加1.默认值为5

multiport

这个模块匹配一组源端口或目标端口,最多可以指定15个端口。只能和-p tcp 或者 -p udp 连着使用。

–source-port [port[, port]]

如果源端口是其中一个给定端口则匹配

–destination-port [port[, port]]

如果目标端口是其中一个给定端口则匹配

–port [port[, port]]

若源端口和目的端口相等并与某个给定端口相等,则匹配。

mark

这个模块和与netfilter过滤器标记字段匹配（就可以在下面设置为使用MARK标记）。

–mark value [/mask]

匹配那些无符号标记值的包（如果指定mask，在比较之前会给掩码加上逻辑的标记）。

owner

此模块试为本地生成包匹配包创建者的不同特征。只能用于OUTPUT链，而且即使这样一些包（如ICMP ping应答）还可能没有所有者，因此永远不会匹配。

–uid-owner userid

如果给出有效的user id，那么匹配它的进程产生的包。

–gid-owner groupid

如果给出有效的group id，那么匹配它的进程产生的包。

–sid-owner seessionid

根据给出的会话组匹配该进程产生的包。

state

此模块，当与连接跟踪结合使用时，允许访问包的连接跟踪状态。

–state state

这里state是一个逗号分割的匹配连接状态列表。可能的状态是:INVALID表示包是未知连接，ESTABLISHED表示是双向传送的连接，NEW表示包为新的连接，否则是非双向传送的，而RELATED表示包由新连接开始，但是和一个已存在的连接在一起，如FTP数据传送，或者一个ICMP错误。

unclean

此模块没有可选项，不过它试着匹配那些奇怪的、不常见的包。处在实验中。

tos

此模块匹配IP包首部的8位tos（服务类型）字段（也就是说，包含在优先位中）。

–tos tos

这个参数可以是一个标准名称，（用iptables -m tos -h 察看该列表），或者数值。

TARGET EXTENSIONS

iptables可以使用扩展目标模块：以下都包含在标准版中。

LOG

为匹配的包开启内核记录。当在规则中设置了这一选项后，linux内核会通过printk()打印一些关于全部匹配包的信息（诸如IP包头字段等）。

–log-level level

记录级别（数字或参看 syslog.conf(5)）。

–log-prefix prefix

在纪录信息前加上特定的前缀：最多14个字母长，用来和记录中其他信息区别。

–log-tcp-sequence

记录TCP序列号。如果记录能被用户读取那么这将存在安全隐患。

–log-tcp-options

记录来自TCP包头部的选项。

–log-ip-options

记录来自IP包头部的选项。

MARK

用来设置包的netfilter标记值。只适用于mangle表。

–set-mark mark

REJECT

作为对匹配的包的响应，返回一个错误的包：其他情况下和DROP相同。

此目标只适用于INPUT、FORWARD和OUTPUT链，和调用这些链的用户自定义链。这几个选项控制返回的错误包的特性：

–reject-with type

Type可以是icmp-net-unreachable、icmp-host-unreachable、icmp-port-nreachable、icmp-proto-unreachable、 icmp-net-prohibited 或者 icmp-host-prohibited，该类型会返回相应的ICMP错误信息（默认是port-unreachable）。选项 echo-reply也是允许的；它只能用于指定ICMP ping包的规则中，生成ping的回应。最后，选项tcp-reset可以用于在INPUT链中,或自INPUT链调用的规则，只匹配TCP协议：将回应一个TCP RST包。

TOS

用来设置IP包的首部八位tos。只能用于mangle表。

–set-tos tos

你可以使用一个数值型的TOS 值，或者用iptables -j TOS -h 来查看有效TOS名列表。

MIRROR

这是一个试验示范目标，可用于转换IP首部字段中的源地址和目标地址，再传送该包,并只适用于INPUT、FORWARD和OUTPUT链，以及只调用它们的用户自定义链。

SNAT

这个目标只适用于nat表的POSTROUTING链。它规定修改包的源地址（此连接以后所有的包都会被影响），停止对规则的检查，它包含选项：

–to-source \[-\]\[:port-port\]

可以指定一个单一的新的IP地址，一个IP地址范围，也可以附加一个端口范围（只能在指定-p tcp 或者-p udp的规则里）。如果未指定端口范围，源端口中512以下的（端口）会被安置为其他的512以下的端口；512到1024之间的端口会被安置为1024以下的，其他端口会被安置为1024或以上。如果可能，端口不会被修改。

–to-destiontion \[-\]\[:port-port\]

可以指定一个单一的新的IP地址，一个IP地址范围，也可以附加一个端口范围（只能在指定-p tcp 或者-p udp的规则里）。如果未指定端口范围，目标端口不会被修改。

MASQUERADE

只用于nat表的POSTROUTING链。只能用于动态获取IP（拨号）连接：如果你拥有静态IP地址，你要用SNAT。伪装相当于给包发出时所经过接口的IP地址设置一个映像，当接口关闭连接会终止。这是因为当下一次拨号时未必是相同的接口地址（以后所有建立的连接都将关闭）。它有一个选项：

–to-ports [-port>]

指定使用的源端口范围，覆盖默认的SNAT源地址选择（见上面）。这个选项只适用于指定了-p tcp或者-p udp的规则。

REDIRECT

只适用于nat表的PREROUTING和OUTPUT链，和只调用它们的用户自定义链。它修改包的目标IP地址来发送包到机器自身（本地生成的包被安置为地址127.0.0.1）。它包含一个选项：

–to-ports []

指定使用的目的端口或端口范围：不指定的话，目标端口不会被修改。只能用于指定了-p tcp 或 -p udp的规则。

DIAGNOSTICS

诊断

不同的错误信息会打印成标准错误：退出代码0表示正确。类似于不对的或者滥用的命令行参数错误会返回错误代码2，其他错误返回代码为1。

BUGS

臭虫

Check is not implemented (yet).

检查还未完成。

COMPATIBILITY WITH IPCHAINS

与ipchains的兼容性

iptables和Rusty Russell的ipchains非常相似。主要区别是INPUT 链只用于进入本地主机的包,而OUTPUT只用于自本地主机生成的包。因此每个包只经过三个链的一个；以前转发的包会经过所有三个链。其他主要区别是 -i 引用进入接口；-o引用输出接口，两者都适用于进入FORWARD链的包。当和可选扩展模块一起使用默认过滤器表时，iptables是一个纯粹的包过滤器。这能大大减少以前对IP伪装和包过滤结合使用的混淆，所以以下选项作了不同的处理：

-j MASQ

-M -S

-M -L

在iptables中有几个不同的链。