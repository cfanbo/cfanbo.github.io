---
title: vmware中freebsd系统同步时间
author: admin
type: post
date: 2010-09-03T07:41:53+00:00
url: /archives/5568
IM_contentdowned:
 - 1
categories:
 - 服务器

---
先设置时区：

\# **tzsetup**

再与国家授时中心服务器对时：
 **\# ntpdate 210.72.145.44**

以后自动同步：

首先修改/etc/rc.conf添加**ntpd_enable=”YES”**到最后一行。

然后配置对时服务器：

**\# vi /etc/ntp.conf**

server 210.72.145.44 prefer
server 159.226.154.47
server 127.127.1.0
fudge 127.127.0.1 stratum 5
restrict default ignore
restrict 127.0.0.0 mask 255.0.0.0
restrict 192.168.0.0 mask 255.255.255.0 noquery nopeer notrust
restrict 210.72.145.44 noquery
restrict 159.226.154.47 noquery
driftfile /var/db/ntpd.drift

/var/run/xntpd.pid

**\# ntpd -p /var/run/ntpd.pid
\# tail /var/log/messages**
Oct 9 16:46:56 chiwawa ntpd[89409]: ntpd 4.1.0-a Thu Apr 3 08:26:24 GMT 2003 (1)
Oct 9 16:46:56 chiwawa ntpd[89409]: kernel time discipline status 2040……
Oct 9 16:50:10 chiwawa ntpd[89409]: time set -0.189546 s

==============================

如果以上的方法不起作用的话,就用下面的办法.

#sysinstall
选择 configure
选择 Time Zone
UTC = no
选择 Asia
选择 China
选择 east China – Beijing,Guangdong,Shanghai etc.
Does the abbreviation \`CST’ look reasonable? = OK
然后退出sysinstall就可以了。

用date命令验证一下
#date
Mon Feb 5 12:35:01 CST 2007

如果时间不对，就用下面的命令同步windows的时间服务器
ntpdate time.windows.com