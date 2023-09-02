---
title: 国内网站 WebServer 和所用 Cache 类型统计
author: admin
type: post
date: 2008-11-12T10:53:31+00:00
excerpt: |
 |
 综合类，从结果上来看 Apache 还是主流：

 Site WebServer Cache
 www.baidu.com BWS/1.0 N/A
 www.qq.com Apache squid/2.6.STABLE5
 www.sina.com.cn Apache/2.0.54 (Unix) N/A
 www.sohu.com Apache/1.3.37 (Unix) mod_gzip/1.3.26.1a squid
 www.163.com Apache/2.2.6 (Unix) N/A
 www.taobao.com Apache N/A
 www.google.cn gws N/A
 www.tom.com Apache NetCache NetApp/6.1.1D4
 www.soso.com Apache N/A
 www.youku.com Apache N/A
 www.xunlei.com Apache/2.2.8 (Unix) N/A
 www.eastmoney.com Microsoft-IIS/6.0 N/A
 www.56.com nginx/0.5.33 squid/2.6.STABLE12-20070426
 www.6.cn nginx/0.6.14 squid/3.0.STABLE1.dev
 www.51.com Apache N/A
 www.yahoo.cn 4EWS N/A
 www.poco.cn Apache N/A
 www.sogou.com Apache/2.0.61 (Unix) Resin/3.0.24
url: /archives/587
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - WebServer

---
综合类，从结果上来看 Apache 还是主流：

Site
 WebServer
 Cache
 [www.baidu.com](http://www.baidu.com/)
 BWS/1.0

 N/A
 [www.qq.com](http://www.qq.com/)
 Apache

 squid/2.6.STABLE5
 [www.sina.com.cn](http://www.sina.com.cn/)
 Apache/2.0.54 (Unix)

 N/A
 [www.sohu.com](http://www.sohu.com/)
 Apache/1.3.37 (Unix) mod_gzip/1.3.26.1a

 squid
 [www.163.com](http://www.163.com/)
 Apache/2.2.6 (Unix)

 N/A
 [www.taobao.com](http://www.taobao.com/)
 Apache

 N/A
 [www.google.cn](http://www.google.cn/)
 gws

 N/A
 [www.tom.com](http://www.tom.com/)
 Apache

 NetCache NetApp/6.1.1D4
 [www.soso.com](http://www.soso.com/)
 Apache

 N/A
 [www.youku.com](http://www.youku.com/)
 Apache

 N/A
 [www.xunlei.com](http://www.xunlei.com/)
 Apache/2.2.8 (Unix)

 N/A
 [www.eastmoney.com](http://www.eastmoney.com/)
 Microsoft-IIS/6.0

 N/A
 [www.56.com](http://www.56.com/)
 nginx/0.5.33

 squid/2.6.STABLE12-20070426
 [www.6.cn](http://www.6.cn/)
 nginx/0.6.14

 squid/3.0.STABLE1.dev
 [www.51.com](http://www.51.com/)
 Apache

 N/A
 [www.yahoo.cn](http://www.yahoo.cn/)
 4EWS

 N/A
 [www.poco.cn](http://www.poco.cn/)
 Apache

 N/A
 [www.sogou.com](http://www.sogou.com/)
 Apache/2.0.61 (Unix)

 Resin/3.0.24

博客类：

Site
 WebServer
 Cache

 blog.sina.com.cn

 nginx/0.5.35

 N/A

 hi.baidu.com

 apache 1.1.23.2

 N/A

 qzone.qq.com

 Apache

 N/A

 blog.sohu.com

 Apache

 squid
 [www.bokee.com](http://www.bokee.com/)
 Apache/1.3.31 (Unix) mod_gzip/1.3.26.1a

 N/A
 [www.blogcn.com](http://www.blogcn.com/)
 Microsoft-IIS/6.0

 Cdn Cache Server V2.0


社区类，这里面相当大的比例用了 M$ IIS，奇怪：

Site
 WebServer
 Cache
 [www.mop.com](http://www.mop.com/)
 lighttpd

 N/A
 [www.cyworld.com.cn](http://www.cyworld.com.cn/)
 Apache

 N/A

 bbs.qq.com

 Apache

 N/A
 [www.tianyaclub.com](http://www.tianyaclub.com/)
 Microsoft-IIS/5.0

 N/A

 bbs.csdn.net

 Microsoft-IIS/6.0

 Longrujun?

 club.xilu.com

 Apache/2.2.0 (Unix) PHP/5.2.1

 N/A
 [www.tiexue.net](http://www.tiexue.net/)
 Microsoft-IIS/6.0

 N/A
 [www.xici.net](http://www.xici.net/)
 Microsoft-IIS/6.0

 N/A

 bbs.sina.com.cn

 Apache/2.0.54 (Unix)

 N/A
 [www.newsmth.net](http://www.newsmth.net/)
 nginx/0.5.35

 squid/3.0.STABLE1


视频类，之所以选择这个类别，是因为视频类对前端要求数据量和流量特别大，连接数特别多，这些特征也能反映出一些问题。从结果看也普遍采用了比较轻快的 lighttpd 或 nginx，另外用 squid 或 cdn 之类做 cache：

Site
 WebServer
 Cache
 [www.6.cn](http://www.6.cn/)
 nginx/0.6.14(网页)

 nginx/0.6.14(视频)

 squid/3.0.STABLE1.dev(网页)
 [www.youku.com](http://www.youku.com/)
 Apache(网页)

 lighttpd(视频)

 N/A
 [www.56.com](http://www.56.com/)
 nginx/0.5.33(网页)

 56.com flv server(视频)

 squid/2.6.STABLE12-20070426(网页)
 [www.ku6.com](http://www.ku6.com/)
 Apache(网页)

 nginx/0.5.35(视频)

 CDN(视频)
 [www.tudou.com](http://www.tudou.com/)
 Apache(网页)

 Cdn Cache Server V2.0(视频)

 chinanetcenter.com(视频)
 [www.pomoho.com](http://www.pomoho.com/)
 Microsoft-IIS/5.0

 N/A


–THE END–