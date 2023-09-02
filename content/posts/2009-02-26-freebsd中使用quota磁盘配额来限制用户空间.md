---
title: FreeBSD中使用QUOTA磁盘配额来限制用户空间
author: admin
type: post
date: 2009-02-26T04:33:55+00:00
excerpt: |
 虚拟主机中经常要限制用户空间的大小和文件的数量。这些限制在linux和FreeBSD中都是用QUOTA来实现的。这里我说下在FreeBSD下实现的方法；
 开启QUOTA支持
 首先需要修改内核加入对quota的支持
 machine i386
 cpu I686_CPU
 #ident GENERIC
 ident CNOSvhost
 maxusers 0
 options QUOTA #就是这行了。
 修改好后重新编译内核。
 然后在/etc/rc.conf里加入：
 enable_quotas="YES"
 check_quotas="YES"
 这样你的系统就起用QUOTA了，你应当通过编辑/etc/fstab的某个文件系统的属性，加入QUOTA的支持。
url: /archives/1070
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - QUOTA

---
虚拟主机中经常要限制用户空间的大小和文件的数量。这些限制在linux和FreeBSD中都是用QUOTA来实现的。这里我说下在FreeBSD下实现的方法；
开启QUOTA支持
首先需要修改内核加入对quota的支持
machine i386
cpu I686_CPU
#ident GENERIC
ident CNOSvhost
maxusers 0
options QUOTA #就是这行了。
修改好后重新编译内核。
然后在/etc/rc.conf里加入：
enable_quotas=_“_YES_“_
check_quotas=_“_YES_“_
这样你的系统就起用QUOTA了，你应当通过编辑/etc/fstab的某个文件系统的属性，加入QUOTA的支持。
下面的fstab文件就设置了在/pub文件系统上起用用户配额和组配额
\# See the fstab(5) manual page for important information on automatic mounts
\# of network filesystems before modifying this file.
\# Device Mountpoint FStype Options Dump Pass#
/dev/ad0s1b none swap sw 0 0
/dev/ad0s1a / ufs rw 1 1
/dev/ad0s1h /pub ufs rw,userquota,groupquota 2 2
/dev/ad0s1e /tmp ufs rw 2 2
/dev/ad0s1g /usr ufs rw 2 2
/dev/ad0s1f /var ufs rw 2 2
/dev/acd0c /cdrom cd9660 ro,noauto 0 0
proc /proc procfs rw 0 0
设置完fstab文件后，执行下面的命令打开quota
\# quotacheck -av
\# repquota -a
基本上前期的工作都已经做完了，剩下的就是编辑用户的配额了。
编辑用户配额
\# edquota c4st将编辑用户c4st的配额设定，出现的是一个文本编辑器界面：
Quotas for user c4st:
/pub: kbytes in use: 3438, limits (soft = 100000, hard = 100020)
inodes in use: 25, limits (soft = 25, hard = 26)
我们看到设定共分为两行。
kbyters in use:3438表示已经使用了3438kb limits限制(soft=100000软限制100M，hard=100020硬限制)
soft表示达到此值时警告，hard表示的用户实际可以使用的大小。
inodes in use: 25, limits (soft = 25, hard = 26)这行为可以拥有的“文件数量”限定，当然例子给出的数值不太实际，上面的设定，
用户只能创建26个文件。实际应用中，可以根据需要调整inodes的值，比如，你要装一个基于文本库的程序，如lb5000（一种webbbs），
一些cgi文章管理等系统，你就要适当的调大inode的hard设定　?
常见的quota命令
\# edquota -t对quota用户使用软限制之前的时间设定，days，hours，minutes或seconds都可以是此设定的单位，值只要是合理就可以。
Time units may be: days, hours, minutes, or seconds
Grace period before enforcing soft limits for users:
/pub: block grace period: 1 day, file grace period: 1 day
\# repquota -a报告文件系统关于quota的信息。
Block limits File limits
User used soft hard grace used soft hard grace
wheel — 2 0 0 – 1 0 0 –
operator — 128 0 0 – 2 0 0 –
mysql — 18 0 0 – 9 0 0 –
vhostuser — 36164 100000 100050 – 342 1000 1005 –
Block limits File limits
User used soft hard grace used soft hard grace
root — 9136 0 0 – 12 0 0 –
mysql — 18 0 0 – 9 0 0 –
testmin — 18 1000 1050 – 9 1000 1005 –
web — 22152 0 0 – 122 0 0 –
coms_cn — 1550 0 0 – 177 0 0 –
c4st -+ 3438 100000 100020 – 25 25 26 22:56
\# Quota　:显示用户的磁盘使用情况和上限。
-g　显示用户所在组的组配额
-l 不显示NFS系统上的配额
-u　显示用户配额
-q　显示使用情况超过配额的简要信息
-v 检查用户的quota设置
\# edquota -p test c4st c4st1 c4st2…..
将把用户test的配额设定复制给用户c4st c4st1 c4st2…..
\# quotacheck -a这个命令可以定期执行，用来检查全部设定是否正常(可以放到crontab里执行)。

**本文来自ChinaUnix博客，如果查看原文请点：** [http://blog.chinaunix.net/u/4206/showart_519282.html](http://blog.chinaunix.net/u/4206/showart_519282.html)