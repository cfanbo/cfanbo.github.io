---
title: PutVarnish on port 80(使varnish工作在 80 端口上)
author: admin
type: post
date: 2011-11-21T04:07:32+00:00
url: /archives/12019
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - varnish

---
**PutVarnish on port 80(使 varnish工作在 80 端口上)**
如果您的程序正常运行，没有问题，我们就可以把varnish调整到80端口运行。先关闭vernish

```
pkill varnishd
```

然后停止您的 web服务器，修改web服务器配置，把 web服务器修改成监听8080端口，然后修改varnish 的default.vcl和改变默认的后端服务器端口为8080.
先启动您的web服务器，然后在启动varnish：

```
varnishd -f /usr/local/etc/varnish/default.vcl -s malloc,1G -T 127.0.0.1:2000
```

我们取消了-a 选项，这样varnish将监控默认端口，启动后，检查您的 web程序是否正常。

**相关教程:**

varnish中Varnishlog命令解析:

linux下varnish配置及使用教程:

Varnish Configuration Language – VCL （varnish 配置 语言-VCL）:

varnish中的Statistics（统计 varnish相关数据）-Varnishtop ,Varnishhist ,Varnishsizes ,Varnishstat: [http://blog.haohtml.com/archives/12036](http://blog.haohtml.com/archives/12036)

bankend Server后端服务高级配置:

varnish中的backends 组Directors: [http://blog.haohtml.com/archives/12049](http://blog.haohtml.com/archives/12049 "varnish中的Directors")

varnish中常用的排错方法: