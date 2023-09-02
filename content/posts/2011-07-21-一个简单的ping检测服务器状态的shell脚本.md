---
title: 一个简单的ping检测服务器状态的shell脚本
author: admin
type: post
date: 2011-07-21T08:14:47+00:00
url: /archives/10551
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ping
 - shell

---
这个脚本特别的简单的,一次只能检测一个ip地址,可以放在crontab里定时检测.可以用来检测服务器状态情况.特别的实用的,如果有多个ip地址的话,可能必定一下,循环一下就可以了.

只有当不通或者宕机后恢复正常的时候才发送指定消息.

```
#!/bin/bash
if [ $# -ne 1 ]
then
echo 'must have one params ip address format!'
exit
fi

ip=$1
tmpfile=$ip.txt
if [ -f $tmpfile ]; then
lastmsg=`cat $tmpfile`
else
lastmsg='YES'
fi

ret=`ping -c 3 $ip | grep ttl  | wc -l`
if [ $ret -lt 2 ]; then
echo 'NO' > $tmpfile
echo 'send waring message!'
//这里可以执行php脚本,用来 发送邮件信息
elif [ $lastmsg = 'NO' ]; then
echo 'YES' > $tmpfile
echo 'send ok message!'
//发送邮件信息
fi
```