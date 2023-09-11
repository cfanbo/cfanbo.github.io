---
title: 高性能JavaScript模板引擎原理解析
author: admin
type: post
date: 2012-11-18T08:40:57+00:00
url: /archives/13497
categories:
 - 前端设计

---

随着 web 发展，前端应用变得越来越复杂，基于后端的 javascript(Node.js) 也开始崭露头角，此时 javascript 被寄予了更大的期望，与此同时 javascript MVC 思想也开始流行起来。javascript 模板引擎作为数据与界面分离工作中最重要一环，越来越受开发者关注，近一年来在开源社区中更是百花齐放，在 Twitter、淘宝网、新浪微博、腾讯QQ空间、腾讯微博等大型网站中均能看到它们的身影。


本文将用最简单的示例代码描述现有的 javascript 模板引擎的原理，包括新一代 javascript 模板引擎 artTemplate 的特性实现原理，欢迎共同探讨。

## artTemplate 介绍

artTemplate 是新一代 javascript 模板引擎，它采用预编译方式让性能有了质的飞跃，并且充分利用 javascript 引擎特性，使得其性能无论在前端还是后端都有极其出色的表现。在 chrome 下渲染效率测试中分别是知名引擎 Mustache 与 micro tmpl 的 25 、 32 倍。


![速度对比](http://cdc.tencent.com/wp-content/uploads/2012/06/1.png)

除了性能优势外，调试功能也值得一提。模板调试器可以精确定位到引发渲染错误的模板语句，解决了编写模板过程中无法调试的痛苦，让开发变得高效，也避免了因为单个模板出错导致整个应用崩溃的情况发生。


artTemplate 这一切都在 1.7kb(gzip) 中实现！


## javascript 模板引擎基本原理

虽然每个引擎从模板语法、语法解析、变量赋值、字符串拼接的实现方式各有所不同，但关键的渲染原理仍然是动态执行 javascript 字符串。


关于动态执行 javascript 字符串，本文以一段模板代码举例：


![](http://cdc.tencent.com/wp-content/uploads/2012/06/code1.png)

这是一段非常朴素的模板写法，其中，”” 为 closeTag (逻辑语句闭合标签)，若 openTag 后面紧跟 “=” 则会输出变量的内容。


HTML语句与变量输出语句被直接输出，解析后的字符串类似：


![](http://cdc.tencent.com/wp-content/uploads/2012/06/code2.png)

语法分析完毕一般还会返回渲染方法：


![](http://cdc.tencent.com/wp-content/uploads/2012/06/code3.png)

渲染测试：


![](http://cdc.tencent.com/wp-content/uploads/2012/06/code4.png)

在上面 render 方法中，模板变量赋值采用了 with 语句，字符串拼接采用数组的 push 方法以提升在 IE6、7 下的性能，jQuery 作者 john 开发的微型模板引擎 tmpl 是这种方式的典型代表，参见： http://ejohn.org/blog/javascript-micro-templating/


由原理实现可见，传统 javascript 模板引擎中留下两个待解决的问题：


1、性能：模板引擎渲染的时候依赖 Function 构造器实现，Function 与 eval、setTimeout、setInterval 一样，提供了使用文本访问 javascript 解析引擎的方法，但这样执行 javascript 的性能非常低下。


2、调试：由于是动态执行字符串，若遇到错误调试器无法捕获错误源，导致模板 BUG 调试变得异常痛苦。在没有进行容错的引擎中，局部模板若因为数据异常甚至可以导致整个应用崩溃，随着模板的数目增加，维护成本将剧增。


## artTemplate 高效的秘密

**1、预编译**

在上述模板引擎实现原理中，因为要对模板变量进行赋值，所以每次渲染都需要动态编译 javascript 字符串完成变量赋值。而 artTemplate 的编译赋值过程却是在渲染之前完成的，这种方式称之为“预编译”。artTemplate 模板编译器会根据一些简单的规则提取好所有模板变量，声明在渲染函数头部，这个函数类似：


![](http://cdc.tencent.com/wp-content/uploads/2012/06/code5.png)

这个自动生成的函数就如同一个手工编写的 javascript 函数一样，同等的执行次数下无论 CPU 还是内存占用都有显著减少，性能近乎极限。


值得一提的是：artTemplate 很多特性都基于预编译实现，如沙箱规范与自定义语法等。


**2、更快的字符串相加方式**

很多人误以为数组 push 方法拼接字符串会比 += 快，要知道这仅仅是 IE6-8 的浏览器下。实测表明现代浏览器使用 += 会比数组 push 方法快，而在 v8 引擎中，使用 += 方式比数组拼接快 4.7 倍。所以 artTemplate 根据 javascript 引擎特性采用了两种不同的字符串拼接方式。


## artTemplate 调试模式原理

前端模板引擎不像后端模板引擎，它是动态解析，所以调试器无法定位到错误行号，而 artTemplate 通过巧妙的方式让模板调试器可以精确定位到引发渲染错误的模板语句，例如：


![debug](http://cdc.tencent.com/wp-content/uploads/2012/06/2.png)

artTemplate 支持两种类型的错误捕获，一是渲染错误(Render Error)与编译错误(Syntax Error)。


**1、渲染错误**

渲染错误一般是因为模板数据错误或者变量错误产生的，渲染的时候只有遇到错误才会进入调试模式重新编译模板，而不会影响正常的模板执行效率。模板编译器根据模板换行符记录行号，编译后的函数类似：


![](http://cdc.tencent.com/wp-content/uploads/2012/06/code6.png)

当执行过程遇到错误，立马抛出异常模板对应的行号，模板调试器再根据行号反查模板对应的语句并打印到控制台。


**2、编译错误**

编译错误一般是模板语法错误，如不合格的套嵌、未知语法等。由于 artTemplate 没有进行完整的词法分析，故无法确定错误源所在的位置，只能对错误信息与源码进行原文输出，供开发者判断。


## 开源节流

artTemplate 基于开源协议发布，无论是商业公司还是个人都可以免费在项目中使用，欢迎共同完善。


**下载地址：**

[https://github.com/aui/artTemplate](https://github.com/aui/artTemplate)

**在线预览：**

[http://aui.github.com/artTemplate/](http://aui.github.com/artTemplate/)

**反馈：**

[腾讯微博](http://t.qq.com/tangbin) | [新浪微博](http://weibo.com/planeart)

 * (本文出自Tencent CDC Blog，转载时请注明出处)