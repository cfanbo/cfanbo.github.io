---
title: JQuery中的ready和js中onload加载优先级问题
author: admin
type: post
date: 2011-03-03T06:00:01+00:00
url: /archives/7943
IM_contentdowned:
 - 1
categories:
 - 其它
tags:
 - js

---

JQuery中的$(function(){})和js中onload，谁先加载？


在$(document).ready()执行时，整个DOM文档树已经解析完成，即各个DOM元素都已经可以访问了（但是对于某些元素的某些属性此时访问可能还不精确，如图片的宽度高度）。$(function(){})在根据需要放置位置，可能在ready之前，可以在其之后。


onload需要页面上所有的资源都加载上之后执行，而ready则是DOM文档树已经解析完成时，说ready比onload快最显著的是比如一个页面上有一个很大的图片，加载要好久，onload只有在图片加载完成之后执行，而ready不必等图片加载完成.


而且一些人所说的向document添加load事件 是完全不可能的。

document准备之前是添加不进onload的，而且在document准备之后加onload是不会执行的。所以，document的onload只在html中声明有时才有用，动态加进去的一般是没用的。

因此要想实现页面加载时的loading效果，一般是在节点里面第一个位置添加div覆盖窗口，然后在或者onload里面把覆盖层移除。

ext的sample就是这么做的，而且把之类的放入body里面也能计算出js文件的加载时间。


$(document).ready()在通常的情况下是限于onload加载的，他是基于DOM形成后就加载相关的应用的，而onload是待整个文档或者说页面加载完成后才会触发的，大家可以做个非常简单的实验，给一个页面加载一张大图片,然后限制下你的网络带宽，页面运用这两种写法alert一个提示。结果就很明显了.


$(document).ready();和$(function(){});是一样的。

$(document).ready()是DOM加载好后就执行，而onload要网页中的内容全部加载后才执行，比如说一个页面里有很多图片，onload就要等这些图片都加载好后才执行。另外一个页面中如果有两个onload只会执行一个，而$(document).ready();可以执行多个.