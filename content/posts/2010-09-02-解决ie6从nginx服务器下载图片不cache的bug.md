---
title: 解决IE6从Nginx服务器下载图片不Cache的Bug
author: admin
type: post
date: 2010-09-02T09:30:26+00:00
url: /archives/5538
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - iebug

---
其实这个Bug是由分两种情况的：

**1.和Nginx无关，是针对CSS背景图片的。**

一般用户不会碰到，更多的时候是开发者将自己的IE的缓存策略从默认的”自动”改为“每次访问都查询”才发生 的。特点是鼠标一旦浮动到有背景图片的地方，IE会不顾已经缓存的图片，自行去服务器再次获取图片，造成图片短暂消失。这个问题比较简单，可以通过以下脚 本解决。

>

> 1
>

**2. 但是实际上更常见的原因是Nginx上打开了Gzip压缩功能。**

这个是IE6 的著名Bug，早在2002年就被人详细讨论过了，在IE7中有所改进，但微软永远也不会去修复IE6了。

根本原因是Nginx对于启用了Gzip的http上下文，即使你在之前的配置文件里声明过 gzip_disable “MSIE [1-6].”，Nginx不再对IE6用Gzip压缩了，但是送出的http报头却仍然采用了和Gzip压缩数据包相匹配的Vary: Accept-Encoding。IE6不认识这个报头，IE6对除了Vary: User-Agent的报头外，都不查询缓存，直接去服务器申请。更绝得是，不是使用查询文件是否更新，而是强行要求一份完整文件。（IE7总算客气了 点，就是先查询一下文件是否更新在下载，减轻了这个bug的影响）。

这个Bug的级别很高，靠前面1中的方法是不行的。在 NGINX中，通过add-header Cache-Control “post-check:3600, pre-check:43200″; ，象这里 介绍的方法， 也是不行的。通过一番痛苦的摸索，才发现要解决这个Bug，有两个方法，一个是使用Nginx的headers-more-nginx-module， 遇到User-Agent是IE6（或者不管三七二十一），修改报头为Vary: User-Agent。不过这个模块并不是标准配置，需要重新编译Nginx。

另一个更为简便易行的方式就是在诸如图片和CSS，JS文件存取的地方加上一下语句，显示关闭Gzip，并手工加上报头。

> 1 if ($http\_user\_agent ~ “MSIE [1-6].”) {
>
> 2 #Must explicitly turns off gzip to prevent Nginx set Vary:Accept-Encoding
>
> 3 #which will prevent IE6 from caching anything
>
> 4 gzip off;
>
> 5 add_header Vary “User-Agent”;
>
> 6 }
>
> 经过我的测试这样是可以解决问题的。这里只能用$http\_user\_agent来获取浏览器参数，曾经试过$ancient_browser，不 过没有成功。

另外，Nginx本身是有一个Gzip的Derective来控制Header的， 就是gzip\_vary。设成gzip\_vary off应该也能解决问题，但这是这个参数只能存在与http, location,server这些上下文里，不能添加到if上下文里，所以只能用gzip off; 把gzip完全给关了。

希望我的血泪可以让你少走一点弯路。难怪每一个做过前端的人都会呐喊，” Let’s Kill IE6 !”。这玩意儿真不是个东西。