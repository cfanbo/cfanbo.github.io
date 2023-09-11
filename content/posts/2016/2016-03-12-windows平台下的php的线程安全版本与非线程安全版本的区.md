---
title: windows平台下的PHP的线程安全版本与非线程安全版本的区别
author: admin
type: post
date: 2016-03-12T06:45:24+00:00
url: /archives/16774
categories:
 - 服务器
tags:
 - php

---
Windows版的PHP从版本5.2.1开始有Thread Safe(线程安全)和None Thread Safe(NTS，非线程安全)之分（Linux/Unit平台没有这个概念的东西的），这两者不同在于何处？到底应该用哪种？这里做一个简单的介绍。

PHP有2中运行方式：**ISAPI**和**FastCGI**。

ISAPI执行方式是以DLL动态库的形式使用，可以在被用户请求后执行，在处理完一个用户请求后不会马上消失，所以需要进行线程安全检查，这样来提高程序的执行效率，所以如果是以ISAPI来执行PHP，建议选择Thread Safe版本；

而FastCGI执行方式是以单一线程来执行操作，所以不需要进行线程的安全检查，除去线程安全检查的防护反而可以提高执行效率，所以，如果是以FastCGI来执行PHP，建议选择Non Thread Safe版本。

对于apache服务器来说一般选择isapi方式，而对于nginx服务器则选择FastCGI方式。

1.**Non Thread Safe**版本php适用在使用CGI以及fastCGI的web服务器上,如nginx,lighttpd以及IIS的CGI模式下

2.**Thread Safe**版本php适用在使用ISAPI或者module的web服务器上,如IIS的ISAPI模式或者apache module模式

这只是一般的适用区别,并不绝对,也就是说两种版本在web服务器上都能使用,并不一定会出什么问题