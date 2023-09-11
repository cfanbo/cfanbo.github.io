---
title: '[原创教程]在FreeBSD下安装BIND,提供dns服务'
author: admin
type: post
date: 2010-09-08T09:06:39+00:00
url: /archives/5616
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - bind
 - dns

---
**一.用ports方式安装bind9**

> #/usr/ports/dns/bind9
> #make install clean

并在/etc/rc.conf文件里添加一行:

> named_enable=”YES”

作为系统服务启动.

**二.配置BIND**

1.编辑/etc/namedb/named.conf 文件,在最下面以下两部分

#正向解析配置文件

> zone “haohtml.com” {
> type master;
> file “master/haohtml.com”;
> };

#反向解析配置文件

> zone “0.168.192.in-addr.arpa” {
> type master;
> file “master/0.168.192.in-addr.arpa”;
> };

然后编辑 listen-on {127.0.0.1;};  的后面添加监听ip地址,如下:

> listen-on {127.0.0.1; 192.168.0.222;};

第个ip后面加一个”;”符号.

对于转发一部分,我们暂不进行配置,这里用不到的.

============================
 2.新建 master/haohtml.com 文件,把下面的内容添加进去

> $TTL 172800
> @       IN      SOA     haohtml.com. root.haohtml.com. (
> 2010090617; Serial
> 172800; Refresh
> 900; Retry
> 3600000; Expire
> 3600); Minimum
> IN      NS      haohtml.com.
> IN      A       192.168.0.222
> www     IN      A       192.168.0.222
> bbs     IN      A       192.168.0.222
> ceshi   IN      A       192.168.0.222

开头的 **@** 代表网域名称 haohtml.com，IN 表示为 internet 的数据型态。
**SOA** 后面接的是**haohtml.com**，表示这台haohtml.com机器是haohtml.com网域中的主要名称服务器。而**root.haohtml.com**表示管理者的Email 是 root@haohtml.com。
正解档中的内容中除了第一行外，每一行的格式为 \[name\] \[ttl\] \[class\] \[type\] [data]。以下是每个字段的说明：
**name**：可以是网域名称或是主机名称，如果不写的话表示与上一个设定相同。
**ttl**：是数据要存活的时间 (time to live)，也就是 cache server 将保留在它的 cache 中的时间。如果不写的话表示和 SOA 中的设定相同。
**class**：指定网络的类型，这个字段应该都是使用 IN 代表 internet。
**type**：设定该笔数据的型态，例如：MX, A, CNAME, PTR, NS 等。
**data**：就是实际设定数据的部份。
**Serial**：这个设定的版本，这次修改的数字必须比上次的数字大，也就是每次修改这个档时，都要将这个数字提高，这样别的服务器才会将数据更新。一般而言，我们会以日期加上几位的数字来表示，如 2004040301 表示 2004 年 4 月 3 日的第一次设定。
**Refresh**：这个数字是次要名称服务器要多久和主要名称服务器比对数据并更新。
**Retry**：如果比对失败，要在几秒后再向主要名称服务器查询。
**Expire**：表示如果次要名称服务器一直连不上主要名称服务器，这笔数据要多久无法比对便失效。这个字段一样是以秒计算。
**Minimum**：表示别的快取服务器可以将你的设定存放多久。

==========================================
 3.编辑反向解析配置文件 master/0.168.192.in-addr.arpa,文件内容如下:

> $TTL    172800
> @       IN      SOA     haohtml.com. root.haohtml.com. (
> 2010091717; Serial
> 172800 ; Refresh
> 900     ; Retry
> 3600000 ; Expire
> 3600 ) ; Minimum
> IN       NS        haohtml.com.
> 222       IN       PTR       haohtml.com.
> 222       IN       PTR       www.haohtml.com.
> 222       IN       PTR       bbs.haohtml.com.
> 222       IN       PTR       ceshi.haohtml.com.

==========================================

**三.配置本机的DNS,并进行测试**

> #vi /etc/reslov.conf

****文件内容修改如下:

> #domain  localhost
> nameserver      192.168.0.222

经过这几的几个步,基本可以使用,以上只是最为基本的配置方法.可以用 nslookup www.haohtml.com 来进行检查配置是否有误.