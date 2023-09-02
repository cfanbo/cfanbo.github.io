---
title: FreeBSD中sysctl命令的使用
author: admin
type: post
date: 2010-12-17T08:14:41+00:00
url: /archives/6982
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - sysctl

---
纪录尝试向你的机器要求你机器未有的服务的connection记录
若你的机器没有跑named 而对方想要向您要求DNS的服务你会看到…

$tail -10 /var/log/message

ohaha /kernel: Connection attempt to TCP 你的IP位置:53 from 对方IP位置:2731

其中2731 乃是只某一个高於1024的high port …

命令：
# sysctl -w net.inet.tcp.log\_in\_vain=1
# sysctl -w net.inet.udp.log\_in\_vain=1

不过这样只有短暂的 重开机就没有了….
所以我们把他写成一个档案放到rc.d 之中…

自动执行：
1.建立档案
/usr/local/etc/rc.d/# vi logstart.sh
(自己取一个格式为*.sh的档案)
内容只有两行…
sysctl -w net.inet.tcp.log\_in\_vain=1
sysctl -w net.inet.udp.log\_in\_vain=1

2.更改权限
chmod 700 logstart.sh

3.执行
./logstart.sh

4.观看结果
tail /var/log/message

你将会发现有许多纪录喔~~可以作为您安全性的参考~~

补充：sysctl会去读/etc/sysctl.conf 这一个档案…
您也可以在这边加上设定~~

特别感谢：Pluto Chang pluto@mole.idv.tw 的补充..^^

明：/etc/rc.sysctl 其中有一部份是这样的..
#
# Read in /etc/sysctl.conf and set things accordingly
#

if [ -f /etc/sysctl.conf ]; then
while read var comments
do
case ${var} in
\#*|”)
;;
*)
sysctl -w ${var}
;;
esac
done < /etc/sysctl.conf
fi