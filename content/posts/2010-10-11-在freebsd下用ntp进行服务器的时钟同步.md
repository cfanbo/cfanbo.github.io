---
title: 在FreeBSD下用NTP进行服务器的时钟同步
author: admin
type: post
date: 2010-10-11T00:40:00+00:00
url: /archives/5957
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - freebs
 - ntpdate

---
使用Network Time Protocol (NTP)来同步服务器的时间的方法如下：

首先在服务器启动的时候需要使用ntpdate一次性的把系统时钟同步过来。在/etc/rc.conf里面加上ntpdate_enable=”YES”就可以在系统启动的时候调用ntpdate进行一次时间同步了。
在rc.conf里面如果没有指定ntpdate_hosts参数的话，ntpdate就会读取/etc/ntp.conf文件里面的server设置。

使用ntpdate同步了时钟以后，还需要通过ntpd来不断监视和调整时钟的正确性。
启动ntpd的方法是在/etc/rc.conf里面加上**ntpd_enable=”YES”**。

ntpdate和ntpd都需要读取/etc/ntp.conf里面的配置信息。最简单的ntp.conf配置文件如下：

server 0.asia.pool.ntp.org
server 1.asia.pool.ntp.org
server 2.asia.pool.ntp.org
server 3.asia.pool.ntp.org

driftfile /var/db/ntp.drift

server命令用来指定internet上的时间服务器地址。driftfile命令用来指定保存时间微调信息的文件。
关 于internet上的始终服务器，可以通过查询http://ntp.isc.org/bin/view/Servers/WebHome获得相关信 息。最简单的时间服务器定义方法是指定服务器所在地区的时间服务器缓冲池。上面的例子里面就可给出了亚洲的服务器缓冲池的定义方法。实际上还可以根据所在 的国家引用单位更小的缓冲池。但是考虑到有些国家的服务器数量很少，还是采用更大一个单位的缓冲池比较可靠。
关于时间服务器缓冲池的相关信息可以通过http://ntp.isc.org/bin/view/Servers/NTPPoolServers查到。

> \# ntpd -p /var/run/ntpd.pid
> \# tail /var/log/messages
> Oct 9 16:46:56 chiwawa ntpd[89409]: ntpd 4.1.0-a Thu Apr 3 08:26:24 GMT 2003 (1)
> Oct 9 16:46:56 chiwawa ntpd[89409]: kernel time discipline status 2040……
> Oct 9 16:50:10 chiwawa ntpd[89409]: time set -0.189546 s

参考资料：
http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-ntp.html
http://www.freebsd.org/cgi/man.cgi?query=ntpdate&sektion=8
http://www.freebsd.org/cgi/man.cgi?query=ntp.conf&sektion=5
http://www.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&apropos=0&manpath=FreeBSD+6.2-RELEASE
http://www.pool.ntp.org/zone/asia

上面的配置方法是服务器可以上网，通过连接internet上的时间服务器对本地时间进行同步的方法。在局域网内如何建立让别的机器可以同步时间的实际服务器的方法还需要调整。