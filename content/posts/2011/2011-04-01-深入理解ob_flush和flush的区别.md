---
title: 深入理解ob_flush和flush的区别
author: admin
type: post
date: 2011-04-01T04:46:22+00:00
url: /archives/8860
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php

---

ob_flush/flush在手册中的描述, 都是刷新输出缓冲区, 并且还需要配套使用, 所以会导致很多人迷惑…

其实, 他们俩的操作对象不同, 有些情况下, flush根本不做什么事情..

ob_*系列函数, 是操作PHP本身的输出缓冲区.所以, ob_flush是刷新PHP自身的缓冲区.

而flush, 严格来讲, 这个只有在PHP做为apache的Module(handler或者filter)安装的时候, 才有实际作用. 它是刷新WebServer(可以认为特指apache)的缓冲区.

在apache module的sapi下, flush会通过调用sapi\_module的flush成员函数指针, 间接的调用apache的api: ap\_rflush刷新apache的输出缓冲区, 当然手册中也说了, 有一些apache的其他模块, 可能会改变这个动作的结果..

> 有些Apache的模块，比如mod_gzip，可能自己进行输出缓存，这将导致flush()函数产生的结果不会立即被发送到客户端浏览器。
>
> 甚至浏览器也会在显示之前，缓存接收到的内容。例如 Netscape浏览器会在接受到换行或 html 标记的开头之前缓存内容，并且在接受到 标记之前，不会显示出整个表格。
>
> 一些版本的 Microsoft Internet Explorer 只有当接受到的256个字节以后才开始显示该页面，所以必须发送一些额外的空格来让这些浏览器显示页面内容。

所以, 正确使用俩者的顺序是. 先ob_flush, 然后flush,当然, 在其他sapi下, 不调用flush也可以, 只不过为了保证你代码的可移植性, 建议配套使用.

 * 作者: laruence( [http://www.laruence.com](http://www.laruence.com/ "风雪之隅"))
 * 本文地址: [http://www.laruence.com/2010/04/15/1414.html][1]
 * 转载请注明出处



 [1]: http://www.laruence.com/2010/04/15/1414.html "Permanet Link to 深入理解ob_flush和flush的区别"