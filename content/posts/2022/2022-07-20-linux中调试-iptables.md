---
title: Linux中调试 iptables
author: admin
type: post
date: 2022-07-20T05:41:24+00:00
url: /archives/31843
toc: true
categories:
 - 服务器
tags:
 - iptables

---
环境：

客户端(windows) 192.168.6.21

服务器(Ubuntu): 192.168.6.23

# 开启iptables调试内核模块 

```
 $ modprobe nf_log_ipv4
 $ sysctl net.netfilter.nf_log.2
 net.netfilter.nf_log.2 = nf_log_ipv4
```

# 添加iptables规则 

```
 $ iptables -t raw -A PREROUTING -p icmp -j TRACE
 $ iptables -t raw -A OUTPUT -p icmp -j TRACE
```

# 测试规则 

客户端执行 ping 命令，

```
 $ ping 192.168.6.23 -n 1
```

这里使用 -n 参数指定发送的包数量为1，方便我们分析日志

此时在服务器上执行查看日志命令, 日志文件为：`/var/log/syslog` 或者 `/var/log/kern.log` 或者 `/var/log/messages`

```
$ tail -f /var/log/syslog
 Jul 20 11:28:40 ubuntu kernel: [ 7606.531051] TRACE: raw:PREROUTING:policy:2 IN=ens37 OUT= MAC=00:0c:29:30:06:44:00:68:eb:c6:60:f2:08:00 SRC=192.168.6.21 DST=192.168.6.23 LEN=60 TOS=0x00 PREC=0x00 TTL=128 ID=33555 PROTO=ICMP TYPE=8 CODE=0 ID=1 SEQ=608
 Jul 20 11:28:40 ubuntu kernel: [ 7606.531146] TRACE: nat:PREROUTING:rule:1 IN=ens37 OUT= MAC=00:0c:29:30:06:44:00:68:eb:c6:60:f2:08:00 SRC=192.168.6.21 DST=192.168.6.23 LEN=60 TOS=0x00 PREC=0x00 TTL=128 ID=33555 PROTO=ICMP TYPE=8 CODE=0 ID=1 SEQ=608
 Jul 20 11:28:40 ubuntu kernel: [ 7606.531192] TRACE: nat:DOCKER:return:3 IN=ens37 OUT= MAC=00:0c:29:30:06:44:00:68:eb:c6:60:f2:08:00 SRC=192.168.6.21 DST=192.168.6.23 LEN=60 TOS=0x00 PREC=0x00 TTL=128 ID=33555 PROTO=ICMP TYPE=8 CODE=0 ID=1 SEQ=608
 Jul 20 11:28:40 ubuntu kernel: [ 7606.531259] TRACE: nat:PREROUTING:policy:2 IN=ens37 OUT= MAC=00:0c:29:30:06:44:00:68:eb:c6:60:f2:08:00 SRC=192.168.6.21 DST=192.168.6.23 LEN=60 TOS=0x00 PREC=0x00 TTL=128 ID=33555 PROTO=ICMP TYPE=8 CODE=0 ID=1 SEQ=608
 Jul 20 11:28:40 ubuntu kernel: [ 7606.531316] TRACE: filter:INPUT:policy:1 IN=ens37 OUT= MAC=00:0c:29:30:06:44:00:68:eb:c6:60:f2:08:00 SRC=192.168.6.21 DST=192.168.6.23 LEN=60 TOS=0x00 PREC=0x00 TTL=128 ID=33555 PROTO=ICMP TYPE=8 CODE=0 ID=1 SEQ=608
 Jul 20 11:28:40 ubuntu kernel: [ 7606.531373] TRACE: nat:INPUT:policy:1 IN=ens37 OUT= MAC=00:0c:29:30:06:44:00:68:eb:c6:60:f2:08:00 SRC=192.168.6.21 DST=192.168.6.23 LEN=60 TOS=0x00 PREC=0x00 TTL=128 ID=33555 PROTO=ICMP TYPE=8 CODE=0 ID=1 SEQ=608
 Jul 20 11:28:40 ubuntu kernel: [ 7606.531424] TRACE: raw:OUTPUT:policy:2 IN= OUT=ens37 SRC=192.168.6.23 DST=192.168.6.21 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=35888 PROTO=ICMP TYPE=0 CODE=0 ID=1 SEQ=608
 Jul 20 11:28:40 ubuntu kernel: [ 7606.531488] TRACE: filter:OUTPUT:policy:1 IN= OUT=ens37 SRC=192.168.6.23 DST=192.168.6.21 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=35888 PROTO=ICMP TYPE=0 CODE=0 ID=1 SEQ=608
```

可以看到除了流量来源 SRC 和目的 DST, 还有一些 ICMP 协议相关的字段，如 TYPE, CODE; 对于 ICMP协议 TYPE 有多类值，其中CODE 根据 TYPE 值的不同而不同。

日志字段

```
Jul 20 11:28:40 ubuntu kernel: [ 7606.531051] TRACE: raw:PREROUTING:policy:2 IN=ens37 OUT= MAC=00:0c:29:30:06:44:00:68:eb:c6:60:f2:08:00 SRC=192.168.6.21 DST=192.168.6.23 LEN=60 TOS=0x00 PREC=0x00 TTL=128 ID=33555 PROTO=ICMP TYPE=8 CODE=0 ID=1 SEQ=608
```

`raw:PREROUTING:policy:2` 这里以”:”分隔四类字段值，分别为 raw 表:PREROUTING 键:policy或 rule : 编号

`IN` 流量流入网卡名称，当流量 为流出时，此字段为空

`OUT` 流量流出网卡名称，当流量为流入时，此字段为空

`MAC` 网卡MAC地址

`SRC` 流量来源IP

`DST` 流量目的IP

`LEN` 数据包大小

`TOS` 服务类型

`ID` 流唯一标识， 如日志中请求ID为 33555, 响应ID为 35888

`PROTO` 数据流协议

`TYPE` 协议ICMP的类型，见下表

`CODE` 协议ICMP类型对应的code

上面日志中第2-7条记录为ping 请求（TYPE=8 CODE=0），而最后两条记录为对ping命令的响应（TYPE=0 CODE=0），由于ping 请求经过了nat 表（PREROUTING）和 filter 两个表的不同链，所有打印多条记录。

**ICMP类型**

| TYPE | CODE | Description | Query | Error |
| ---- | ---- | -------------------------------------------------------------------- | ----- | ----- |
| | | Echo Reply——回显应答（Ping应答） | x | |
| 3 | | Network Unreachable——网络不可达 | | x |
| 3 | 1 | Host Unreachable——主机不可达 | | x |
| 3 | 2 | Protocol Unreachable——协议不可达 | | x |
| 3 | 3 | Port Unreachable——端口不可达 | | x |
| 3 | 4 | Fragmentation needed but no frag. bit set——需要进行分片但设置不分片比特 | | x |
| 3 | 5 | Source routing failed——源站选路失败 | | x |
| 3 | 6 | Destination network unknown——目的网络未知 | | x |
| 3 | 7 | Destination host unknown——目的主机未知 | | x |
| 3 | 8 | Source host isolated (obsolete)——源主机被隔离（作废不用） | | x |
| 3 | 9 | Destination network administratively prohibited——目的网络被强制禁止 | | x |
| 3 | 10 | Destination host administratively prohibited——目的主机被强制禁止 | | x |
| 3 | 11 | Network unreachable for TOS——由于服务类型TOS，网络不可达 | | x |
| 3 | 12 | Host unreachable for TOS——由于服务类型TOS，主机不可达 | | x |
| 3 | 13 | Communication administratively prohibited by filtering——由于过滤，通信被强制禁止 | | x |
| 3 | 14 | Host precedence violation——主机越权 | | x |
| 3 | 15 | Precedence cutoff in effect——优先中止生效 | | x |
| 4 | | Source quench——源端被关闭（基本流控制） | | |
| 5 | | Redirect for network——对网络重定向 | | |
| 5 | 1 | Redirect for host——对主机重定向 | | |
| 5 | 2 | Redirect for TOS and network——对服务类型和网络重定向 | | |
| 5 | 3 | Redirect for TOS and host——对服务类型和主机重定向 | | |
| 8 | | Echo request——回显请求（Ping请求） | x | |
| 9 | | Router advertisement——路由器通告 | | |
| 10 | | Route solicitation——路由器请求 | | |
| 11 | | TTL equals 0 during transit——传输期间生存时间为0 | | x |
| 11 | 1 | TTL equals 0 during reassembly——在数据报组装期间生存时间为0 | | x |
| 12 | | IP header bad (catchall error)——坏的IP首部（包括各种差错） | | x |
| 12 | 1 | Required options missing——缺少必需的选项 | | x |
| 13 | | Timestamp request (obsolete)——时间戳请求（作废不用） | x | |
| 14 | | Timestamp reply (obsolete)——时间戳应答（作废不用） | x | |
| 15 | | Information request (obsolete)——信息请求（作废不用） | x | |
| 16 | | Information reply (obsolete)——信息应答（作废不用） | x | |
| 17 | | Address mask request——地址掩码请求 | x | |
| 18 | | Address mask reply——地址掩码应答 | x | |

在日志里同时还有 raw表的 PREROUTING 和 OUTPUT 链的相关记录。

现在我们再添加一条禁止ICMP的规则，这里即可以在filter 表中的 INPUT 链中添加，也可以在 OUTPUT 链中添加。

```
 $ iptables -t filter -A OUTPUT -d 192.168.6.21 -j DROP
```

这里我们添加在了 OUTPUT 链里，所以这里使用的 -d 参数值为 192.168.6.21

现在我们再看一下日志输出

```
Jul 20 11:09:58 ubuntu kernel: [ 6484.565458] TRACE: raw:PREROUTING:policy:2 IN=ens37 OUT= MAC=00:0c:29:30:06:44:00:68:eb:c6:60:f2:08:00 SRC=192.168.6.21 DST=192.168.6.23 LEN=60 TOS=0x00 PREC=0x00 TTL=128 ID=33554 PROTO=ICMP TYPE=8 CODE=0 ID=1 SEQ=582
 Jul 20 11:09:58 ubuntu kernel: [ 6484.565548] TRACE: nat:PREROUTING:rule:1 IN=ens37 OUT= MAC=00:0c:29:30:06:44:00:68:eb:c6:60:f2:08:00 SRC=192.168.6.21 DST=192.168.6.23 LEN=60 TOS=0x00 PREC=0x00 TTL=128 ID=33554 PROTO=ICMP TYPE=8 CODE=0 ID=1 SEQ=582
 Jul 20 11:09:58 ubuntu kernel: [ 6484.565592] TRACE: nat:DOCKER:return:3 IN=ens37 OUT= MAC=00:0c:29:30:06:44:00:68:eb:c6:60:f2:08:00 SRC=192.168.6.21 DST=192.168.6.23 LEN=60 TOS=0x00 PREC=0x00 TTL=128 ID=33554 PROTO=ICMP TYPE=8 CODE=0 ID=1 SEQ=582
 Jul 20 11:09:58 ubuntu kernel: [ 6484.565631] TRACE: nat:PREROUTING:policy:2 IN=ens37 OUT= MAC=00:0c:29:30:06:44:00:68:eb:c6:60:f2:08:00 SRC=192.168.6.21 DST=192.168.6.23 LEN=60 TOS=0x00 PREC=0x00 TTL=128 ID=33554 PROTO=ICMP TYPE=8 CODE=0 ID=1 SEQ=582
 Jul 20 11:09:58 ubuntu kernel: [ 6484.565673] TRACE: filter:INPUT:policy:1 IN=ens37 OUT= MAC=00:0c:29:30:06:44:00:68:eb:c6:60:f2:08:00 SRC=192.168.6.21 DST=192.168.6.23 LEN=60 TOS=0x00 PREC=0x00 TTL=128 ID=33554 PROTO=ICMP TYPE=8 CODE=0 ID=1 SEQ=582
 Jul 20 11:09:58 ubuntu kernel: [ 6484.565713] TRACE: nat:INPUT:policy:1 IN=ens37 OUT= MAC=00:0c:29:30:06:44:00:68:eb:c6:60:f2:08:00 SRC=192.168.6.21 DST=192.168.6.23 LEN=60 TOS=0x00 PREC=0x00 TTL=128 ID=33554 PROTO=ICMP TYPE=8 CODE=0 ID=1 SEQ=582
 Jul 20 11:09:58 ubuntu kernel: [ 6484.565763] TRACE: raw:OUTPUT:policy:2 IN= OUT=ens37 SRC=192.168.6.23 DST=192.168.6.21 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=14584 PROTO=ICMP TYPE=0 CODE=0 ID=1 SEQ=582
 Jul 20 11:09:58 ubuntu kernel: [ 6484.565804] TRACE: filter:OUTPUT:rule:1 IN= OUT=ens37 SRC=192.168.6.23 DST=192.168.6.21 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=14584 PROTO=ICMP TYPE=0 CODE=0 ID=1 SEQ=582
```

请求ID为 33554，TYPE=8, 而响应ID为 14584，TYPE=0

我们看下最后一条日志，其中`filter:OUTPUT:rule:1` 表示路径为 `filter` 表的 `OUTPUT` 链中的编号为`1`的规则，这条应该是我们上面添加的规则 ，我们现在确认一下

```
 iptables -t filter -L OUTPUT -nv --line-number
 Chain OUTPUT (policy ACCEPT 65 packets, 7466 bytes)
 num   pkts bytes target     prot opt in     out     source               destination
 1        3   180 DROP       all  --  *      *       0.0.0.0/0            192.168.6.21
```

这里是添加在了OUTPUT链中，所以 dst 就是客户端的ip地址，src 就是服务器的地址，这个与我们在 INPUT 链中的正好相反。

# 清理现场 

```
$ modprobe -r nf_log_ipv4
modprobe: FATAL: Module nf_log_syslog is in use.
```

# 参考资料 

* https://www.ibm.com/docs/en/qsip/7.4?topic=applications-icmp-type-code-ids
* https://www.rfc-editor.org/rfc/rfc792.html