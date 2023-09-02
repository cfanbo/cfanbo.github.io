---
title: addEventListener的参数说明
author: admin
type: post
date: 2009-05-21T03:48:48+00:00
excerpt: |
 |
 我想大家对这个函数的前两个参数已经很了解了吧，主要是第三个参数不很好理解。我查了一些资料，弄明白了这个问题，所以记录下来了。下面的内容，基本上是参考别人的。

 第三个参数叫做useCapture，是一个boolean值，就是true or false，如果是rue的话就是浏览器会使用Capture方式，false的话是Bubbling，只有在特定状况下才会有影响，通常建议是false，而会有影响的情形是目标元素(target element)有父元素(ancestor element)，而且也有同样的事件对应函数，我想，看图比较清楚。
url: /archives/1429
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - javascript

---

我想大家对这个函数的前两个参数已经很了解了吧，主要是第三个参数不很好理解。我查了一些资料，弄明白了这个问题，所以记录下来了。下面的内容，基本上是参考别人的。

第三个参数叫做useCapture，是一个boolean值，就是true or false，如果是rue的话就是浏览器会使用Capture方式，false的话是Bubbling，只有在特定状况下才会有影响，通常建议是false，而会有影响的情形是目标元素(target element)有父元素(ancestor element)，而且也有同样的事件对应函数，我想，看图比较清楚。


[![](http://blog.haohtml.com/wp-content/uploads/2009/05/f781ff0f5563e6386159f31f.jpg)](/wp-content/uploads/2009/05/f781ff0f5563e6386159f31f.jpg)

像這張圖所顯示的，我的範例有兩層div元素，而且都設定有click事件，一般來說，如果我在內層藍色的元素上click不只會觸發藍色元素的click事件，還會同時觸發紅色元素的click事件，而useCapture這個參數就是在控制這時候兩個click事件的先後順序。如果是false，那就會使用bubbling，他是從內而外的流程，所以會先執行藍色元素的click事件再執行紅色元素的click事件，如果是true，那就是capture，和bubbling相反是由外而內，會先執行紅色元素的click事件才執行藍色元素的click事件。


那如果不同層的元素使用的useCapture不同呢？就是會先從最外層元素往目標元素尋找設定為capture的事件，到達目標元素執行目標元素的事件後，再尋原路往外尋找設定為bubbling的事件。