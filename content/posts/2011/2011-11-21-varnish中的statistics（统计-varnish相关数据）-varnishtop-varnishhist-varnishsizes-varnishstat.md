---
title: varnish中的Statistics（统计 varnish相关数据）-Varnishtop ,Varnishhist ,Varnishsizes ,Varnishstat
author: admin
type: post
date: 2011-11-21T04:30:31+00:00
url: /archives/12036
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - varnish
 - Varnishhist
 - Varnishsizes
 - Varnishstat
 - Varnishtop

---
现在您的varnish已经正常运行，我们来看一下varnish在做什么，这里有些工具可以帮助您做到。
**Varnishtop**
Varnishtop工具读取共享内存的日志，然后连续不断的显示和更新大部分普通日志。
适当的过滤使用 –I，-i，-X 和-x 选项，它可以按照您的要求显示请求的内容，客户端，浏览器等其他日志里的信息。

> varnishtop -i rxurl \\您可以看到客户端请求的 url次数。
> Varnishtop -i txurl \\您可以看到请求后端服务器的url次数。
> Varnishtop -i Rxheader –I Accept-Encoding \\可以看见接收到的头信息中有有多少次包含Accept-Encoding。

**Varnishhist**
Varnishhist工具读取varnishd的共享内存段日志，生成一个连续更新的柱状图，显示最后 N 个请求的处理情况。这个 N 的值是终端的纵坐标的高度，横坐标代表的是对数，如果缓存命中就标记“|”，如果缓存没有命中就标记上“#”符号。

**Varnishsizes**
Varnishsizes 和varnishhist相似，除了varnishsizes现实了对象的大小，取消了完成请求的时间。这样可以大概的观察您的服务对象有多大。

**Varnishstat**
Varnish 有很多计数器，我们计数丢失率，命中率，存储信息，创建线程，删除对象等，几乎所有的操作。Varnishstat将存储这些数值，在优化varnish的时候使用这个命令。

有一个程序可以定期轮询 varnishstat 的数据并生成好看的图表。这个项目叫做Munin。 Munin可以在 http://munin-monitoring.org/找到。在varnish的源码中有munin插件。