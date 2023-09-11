---
title: '使用Nginx轻松实现开源负载均衡──9 月20日在ChinaUnix技术沙龙上的演讲PPT[原创]'
author: admin
type: post
date: 2010-04-01T15:31:14+00:00
url: /archives/3213
IM_data:
 - 'a:1:{s:56:"http://blog.s135.com/template/RuiPai/images/download.gif";s:68:"http://blog.haohtml.com/wp-content/uploads/2011/03/f896_download.gif";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx

---
[文章作者：张宴 本文版本：v1.0 最后修改：2008.09.21 转载请注明原文链接： [http://blog.s135.com/post/369/](http://blog.s135.com/post/369/)]

9月20日下午，我应邀参加了 [ChinaUnix](http://www.chinaunix.net/) 举办的以“如何搞定服务器负载均衡？”为主题的技术沙龙（ [http://linux.chinaunix.net/bbs/thread-1019366-1-1.html](http://linux.chinaunix.net/bbs/thread-1019366-1-1.html)）， 很高兴能够跟诸多业界精英一起探讨交流，很荣幸能够与Unix资深系统工程师──田逸、HonestQiao，以及F5资深技术工程师──杨明非，同台演 讲。

[![chinaunix](http://blog.haohtml.com/wp-content/uploads/2010/04/chinaunix.gif)][1]

* * *

《使用Nginx轻松实现开源负载均衡》是我的演讲PPT（PowerPiont），现提供下载。

**PPT分为四个 部分：**
1、介绍Nginx的基本特征，以及使用Nginx做负载均衡器的理由。

2、用实例，来介绍 Nginx负载均衡在大型网站的典型应用。

3、以实现网站动静分离为原型，对NetScaler硬件七层负载均衡和Nginx软件负 载均衡做一个对比。

①、NetScaler负载均衡交换机动静分离系统架构图
[![netscaler_lb](http://blog.haohtml.com/wp-content/uploads/2010/04/netscaler_lb.png)][2]

②、Nginx反向代理负载均衡器动静分离系统架构图
[![nginx_lb](http://blog.haohtml.com/wp-content/uploads/2010/04/nginx_lb.png)][3]

③、PHP利用Memcached实现session共享，程序无需作任何修改：
修改php.ini（需要memcache.so扩展）

session.save_handler = memcache

session.save_path = tcp://192.168.1.2:11211

4、介绍如何亲自动手，按照步骤，在“五分钟内搞定 Nginx 负载均衡”。

* * *

**PPT下载：**

![](http://blog.s135.com/template/RuiPai/images/download.gif)下载 文件


[点击这里下载文件](http://blog.s135.com/attachment/200809/nginx_lb.zip)

[![chinaunix_zy](http://blog.haohtml.com/wp-content/uploads/2010/04/chinaunix_zy.jpg)][4]

 [1]: http://blog.haohtml.com/wp-content/uploads/2010/04/chinaunix.gif
 [2]: http://blog.haohtml.com/wp-content/uploads/2010/04/netscaler_lb.png
 [3]: http://blog.haohtml.com/wp-content/uploads/2010/04/nginx_lb.png
 [4]: http://blog.haohtml.com/wp-content/uploads/2010/04/chinaunix_zy.jpg