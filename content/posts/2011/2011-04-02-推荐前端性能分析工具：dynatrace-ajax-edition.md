---
title: DynaTrace Ajax Edition：IE浏览器性能分析工具
author: admin
type: post
date: 2011-04-02T03:58:41+00:00
url: /archives/8929
IM_data:
 - 'a:3:{s:66:"http://blog.dynatrace.com/wp-content/step4_summaryview-600x433.PNG";s:85:"http://blog.haohtml.com/wp-content/uploads/2011/04/8338_step4_summaryview-600x433.PNG";s:73:"http://blog.dynatrace.com/wp-content/step6_drillintotimeframe-600x343.PNG";s:92:"http://blog.haohtml.com/wp-content/uploads/2011/04/e742_step6_drillintotimeframe-600x343.PNG";s:71:"http://blog.dynatrace.com/wp-content/step6_dynamicscripttag-600x343.PNG";s:90:"http://blog.haohtml.com/wp-content/uploads/2011/04/c849_step6_dynamicscripttag-600x343.PNG";}'
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - dynaTrace
 - 前端优化

---
DynaTrace AJAX是一个运行在IE浏览器下的免费页面性能分析工具，它可以支持主流的IE6、IE7、IE8浏览器。这款工具正是DynaTrace为进入前端性能分析领域而发布的。您可以利用它来分析页面渲染时间、DOM方法执行时间，甚至可以看到JS代码的解析时间。连JQuery的创始者 John Resig 也鼎力推荐了一把。

从John Resig的 [Deep Tracing of Internet Explorer](http://ejohn.org/blog/deep-tracing-of-internet-explorer/) 了解到了这款刚发布的免费的前端性能分析工具，John Resig对其评价甚高,John Resig对其评价到：“我一般不随便写关于性能分析工具的东西，坦率地说，我感觉它们绝大部分都比较烂，根本不能提供任何有价值的信息和分析结果。不过 dynaTrac提供了一些我以前在任何其他工具上都没见过的东西。”

Ajax的本事真不是盖的！那么，它到底有啥特别之处呢？“这个工具可以跟踪JavaScript从执行开始，经过本地的XMLHttpRequest、发送网络请求，再到请求返回的全过程。”

**更多的我这里就不多说了，权威人士们都说过了，这东西我也刚上手没多久，还谈不上有多深入的技巧可以分享，这里只是先挖个坑，喜欢的就**

**自愿往里跳****吧！**

![](http://blog.dynatrace.com/wp-content/step4_summaryview-600x433.PNG)![](http://blog.dynatrace.com/wp-content/step6_drillintotimeframe-600x343.PNG)![](http://blog.dynatrace.com/wp-content/step6_dynamicscripttag-600x343.PNG)

一旦您下载并安装了DynaTrace Ajax Edition, 您必须进入开始菜单里面的程序组，找到DynaTrace。很明显，首先要做的是录入一个url链接，接下来，点击播放图标的按钮，选择“New Run Configuration”，录入一个新的URL.

DynaTrace AJAX的特点之一是它可以运行在多页面的工作流之下，你可以输入起始网址，然后导航到其他网页或启动Ajax特性，而DynaTrace AJAX在后台监视一切。当您关闭IE浏览器时，您就可以分析所有DAE收集的信息了.

DynaTrace AJAX区别于其它工具的主要特征： 深入分析JavaScript。通过检测事件触发和JavaScript API调用，时间线被分割成不同部分。它包含了HTTP瀑布图。另一个特征是可以保存DynaTrace AJAX分析结果，这样你可以事后检查并且和同事分享它。它还有一些其它很有趣的特征，例如，自动将精简后的源码格式化，这样你可以在现场调试精简代码时，查看更易懂的版本，你还可以分析CPU占用和页面渲染性能

当需要分析JavaScript引起的性能问题时，DynaTrace Ajax Edition包含了从高级调用到实际执行的代码详细信息，你可以查看到底是哪一行JavaScript代码导致了页面的性能瓶颈。我建议你测试一下这个工具并将它添加到你的性能测试工具包之中。

**相关资源：**

下载安装：** [dynaTrace AJAX Edition](http://ajax.dynatrace.com/pages/download/download.aspx)**

这个小视频是个很好的开始： [**5 Minutes Demo >**](http://ajax.dynatrace.com/pages/learn/teaser.aspx)
除了上面John Resig那篇文章，这篇 [**A Step-by-Step Guide to dynaTrace Ajax Edition**](http://blog.dynatrace.com/2009/11/17/a-step-by-step-guide-to-dynatrace-ajax-edition-available-today-for-public-download/ "Permanet Link to A Step-by-Step Guide to dynaTrace Ajax Edition, available today for public download") 是绝对不能错过的！

**其他：**

1. 这个工具是完全免费的，它还有一个孪生兄弟：dynaTrace Diagnostics，专门是用来跟踪服务器端性能的，两兄弟如果联合起来使用，就可以从头到尾，从前到后无所不能了！不过！！！那哥们是收银子的，而且估计身价不低。

2.官方已经承诺，未来会支持Firefox。