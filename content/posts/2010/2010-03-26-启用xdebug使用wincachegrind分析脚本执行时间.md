---
title: 启用Xdebug使用WinCacheGrind分析脚本执行时间
author: admin
type: post
date: 2010-03-26T07:37:15+00:00
excerpt: |
 使用Xdebug调试和优化PHP程序系列教程之WinCacheGrind，教你如何利用Xdebug 配合WinCacheGrind工具来检测PHP代码的效率以及分析PHP代码。

 有时候代码没有明显的编写错误，没有显示任何错误信息（如 error、warning、notice等），但是这不表明代码就是正确无误的。有时候可能某段代码执行时间过长，占用内存过多以致于影响整个系统的效 率，我们没有办法直接看出来是哪部份代码出了问题。这时候我们希望把代码的每个阶段的运行情况都监控起来，写到日志文件中去，运行一段时间后再进行分析， 找到问题所在。

 回忆一下，之前我们编辑php.ini文件
url: /archives/3088
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - php
 - WinCacheGrind
 - xdebug

---

使用Xdebug调试和优化PHP程序系列教程之WinCacheGrind，教你如何利用Xdebug 配合WinCacheGrind工具来检测PHP代码的效率以及分析PHP代码。

另外还有一个结果分析展示工具webgrind。可参考： [http://blog.sina.com.cn/s/blog_635833b3010127q5.html](http://blog.sina.com.cn/s/blog_635833b3010127q5.html)

有时候代码没有明显的编写错误，没有显示任何错误信息（如 error、warning、notice等），但是这不表明代码就是正确无误的。有时候可能某段代码执行时间过长，占用内存过多以致于影响整个系统的效 率，我们没有办法直接看出来是哪部份代码出了问题。这时候我们希望把代码的每个阶段的运行情况都监控起来，写到日志文件中去，运行一段时间后再进行分析， 找到问题所在。

回忆一下，之前我们编辑php.ini文件
加入

`[Xdebug]<br />
xdebug.profiler_enable=on<br />
xdebug.trace_output_dir="I:\Projects\xdebug"<br />
xdebug.profiler_output_dir="I:\Projects\xdebug" `

这几行，目的就在于把执行情况的分析文件写入到”I:\Projects\xdebug”目录中去 （你可以替换成任何你想设定的目录）。如果你执行某段程序后，再打开相应的目录，可以发现生成了一堆文件，例如 cachegrind.out.1169585776这种格式命名的文件。这些就是 Xdebug生成的分析文件。用编辑器打开你可以看到很多程序运行的相关细节信息，不过很显然这样看太累了，我们需要用图形化的软件来查看。

安装教程也可以参考这里： [http://blog.haohtml.com/index.php/archives/3096](http://blog.haohtml.com/index.php/archives/3096)

**WinCacheGrind 下载**
在Windows平台下，可以用 WinCacheGrind(wincachegrind.souceforge.net)这个软件来打开这些文件。可以直观漂亮地显示其中内容：
[![win_xdebug_WinCacheGrind_1](http://blog.haohtml.com/wp-content/uploads/2010/03/win_xdebug_WinCacheGrind_1.jpg)][1]
哇，非常漂亮，我们很直观地看到 index.php中我们调用了一个函数testXdebug()，testXdebug()中又调用了requireFile()函数。这样我们就可以 非常方便地查看整个脚本的程序结构。
另外，我们还可以看到每个函数被调用的次数及执行所花费的时间！这对于测试程序性能非常有用。
[![win_xdebug_WinCacheGrind_2](http://blog.haohtml.com/wp-content/uploads/2010/03/win_xdebug_WinCacheGrind_2.jpg)][2]
好了，这么一个简单的程序不太能 显示出Xdebug+WinCacheGrind的强大，我给出一个稍大点的例子（一个基于Zend Framework的CMS的index.php）：
[![win_xdebug_WinCacheGrind_3](http://blog.haohtml.com/wp-content/uploads/2010/03/win_xdebug_WinCacheGrind_3.jpg)][3]
从上图可以看到：整个程序的结构， 每个函数被调用的次数，执行时间都一目了然。
[![win_xdebug_WinCacheGrind_4](http://blog.haohtml.com/wp-content/uploads/2010/03/win_xdebug_WinCacheGrind_4.jpg)][4]
**WinCacheGrind 小结**：
Xdebug提供了各种自带的函数，并对已有的某些PHP函数进行覆写，可以方便地用于调试排错；Xdebug还可以跟 踪程序的运行，通过对日志文件的分析，我们可以迅速找到程序运行的瓶颈所在，提高程序效率，从而提高整个系统的性能。

**Self**是代表此Funcion自己花费的时间，不包含此Function调用的其他Function。

**Cum**则是此Funcion整体花费的时间，包含此Function调用的其他Function。

 [1]: /wp-content/uploads/2010/03/win_xdebug_WinCacheGrind_1.jpg
 [2]: http://blog.haohtml.com/wp-content/uploads/2010/03/win_xdebug_WinCacheGrind_2.jpg
 [3]: http://blog.haohtml.com/wp-content/uploads/2010/03/win_xdebug_WinCacheGrind_3.jpg
 [4]: http://blog.haohtml.com/wp-content/uploads/2010/03/win_xdebug_WinCacheGrind_4.jpg