---
title: float margin-left 浮动对象双倍距离
author: admin
type: post
date: 2010-09-01T04:19:58+00:00
url: /archives/5407
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - css
 - iebug

---
出现问题是：使用 float: left; 后，在IE显示margin-left:1px;就变成2px的距离。
IE Bug 的解决方法：
加一个 display: inline; 就OK了

> ```
> #box1{
> 	float: left;
> 	background: #F2F2F2;
> 	width: 300px;
> 	height: 200px;
> 	margin-left: 50px;
> }
> ```

> ```
> #box1{
> 	float: left;
> 	background: #F2F2F2;
> 	width: 300px;
> 	height: 200px;
> 	margin-left: 50px;
> 	display: inline;
> }
> ```

margin在IE6下被解释为双倍距离，出现了Margin与float一起用时，在IE6下，其Margin属性会被解释会双倍的距离，margin产生双倍距离其解决兼容问题的两种方法：

1、给当前层增加display: inline;属性。

2、取消浮动：Float。


熟悉规则的人知道浮动元素自动设置为”block”元素，而不管他们之前是什么。这说明浮动元素上的{display: inline;}会被忽略，事实上所有的浏览器没有呈现任何改变，包括IE。但是，它不知何故让IE停止将浮动元素的边界翻倍。因而，这个修复办法在目前 的浏览器下可以被直接应用，而没有任何繁琐的隐藏方法。

应用浮动+margin在IE6会出现双倍距离的现象，注意 只是出现在IE6中，通常这种现象可以用_margin来解决，即针对IE6的hack。 display问题，并不是只有两种显示方式inline和block，还有许多其它的display值，譬如inline-block，可以试一试。在 应用了float之后，无论块级元素还是行内元素都会宽度自适了（当然你没有设定宽度的情况下），就形似inline-block。