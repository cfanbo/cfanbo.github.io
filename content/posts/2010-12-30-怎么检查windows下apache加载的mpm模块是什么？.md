---
title: 怎么检查windows下apache加载的mpm模块是什么？
author: admin
type: post
date: 2010-12-30T04:17:39+00:00
url: /archives/7363
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache

---
现在有很多php运行环境都apache等都用在windows主机上了，但是性能和linux上的应该有些差。于是有很多优化windows下apache性能。优化apache加载mpm是必不可少的一环。

怎么检查自己的windows服务器中apache加载的mpm模块是什么呢？

其实很简单：

“开始-运行-cmd” 打开命令提示符

执行”httpd -l”就可以了。

[![](http://blog.haohtml.com/wp-content/uploads/2010/12/windows_apache_modules.gif)][1]

 [1]: http://blog.haohtml.com/wp-content/uploads/2010/12/windows_apache_modules.gif