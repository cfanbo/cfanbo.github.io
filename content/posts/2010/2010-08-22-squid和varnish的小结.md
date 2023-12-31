---
title: squid和varnish的小结
author: admin
type: post
date: 2010-08-22T03:46:44+00:00
url: /archives/5223
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - Squid
 - varnish

---
上周初步接触linux下的这2个反向缓存软件，都实验了一下，貌似squid还是比较顺利的，varnish则碰到了一些问题

从varnish的文档看，性能比squid强很多，而且不是一点点，下面国外某在线媒体的12台squid换成3台varnish前后访问响应延时比较，据说有人也测试过的确如此，

[![](https://blogstatic.haohtml.com//uploads/2023/09/squid-vs-varnish.jpg)][1]

但我就不那么顺利了。先说squid，安装很顺利，网上的中文文档也很多，在这次尝试中，被缓存的网站的静态内容并不多，主要还是以PHP为主，所以反向cache的效果并不是很好，缓存命中率在60到70%之间，缓存的主要对象是图片。由于安装调试都很顺利，所以在“试玩”了一天后，直接就上线用上了，几天下来，正常。由于做反向缓存的服务器内存不大，只有1G，所以缓存大小只设置了384M，使用了shm，保证了速度，但应该是没有充分发挥出缓存的效能，后期准备增加1G的内存，把缓存扩大到1G，这样的话，缓存对象的大小还能再设置大一些，也许有些mp3之类的也可以缓存起来，这样命中率也许会更高些。

但即便缓存的命中不是很高，但对后面WEB服务器的压力确实是大大减少了，下面是web服务器上并发连接的图示，一处有个跌落，是架squid的时候，二处应该是个异常，上线3天来，WEB服务上的并发连接数明显少于以前。

[![](https://blogstatic.haohtml.com//uploads/2023/09/squid-08211148.jpg)][2]

再说varnish，安装应该还是顺利的，但文档少，跑起来以后发现缓存了比不缓存还要慢很多，一开始以为测试用的Linux内核比较低（要求是freebsd或者linux2.6内核，虽然我的测试环境也满足要求，但还只是2.6的比较早的版本），但后来发现在EL4U5上也一样慢，后来又怀疑自己安装的方式不对，于是折腾了找了比较新的autoconf和automake，但还是老样子。上网查到也有人和我碰到一样的问题，但没有解决方法，毕竟是个新东东，用的人很少。看E文的文档，也没有太大的收获。

最近身体有些问题，状态很不好，我又是个破罐子破摔的，晚上索性就往里面钻了一把：在被varnish缓存的网站和IE客户端都抓包分析，看到底为什么慢。抓包分析也不是轻松的事情，但最终在我眼花缭乱的时候，发现了问题所在：如果WEB服务器返回给缓存服务器的是not modified，那varnish就抓瞎了，既不去取，也不告诉IE什么。可能是程序的问题，也可能是我的配置问题，但还没有时间去研究，日后再说。今天上午只是简单地禁用了WEB服务端上的过期设置，varnish就基本正常了。

Varnish\_cache\_原理篇: [http://docs.haohtml.com/download/cdn/Varnish_cache_%d4%ad%c0%ed%c6%aa.pdf 
