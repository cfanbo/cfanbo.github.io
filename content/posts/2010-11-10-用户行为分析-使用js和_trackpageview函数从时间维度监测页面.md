---
title: 用户行为分析-使用JS和_trackPageview函数从时间维度监测页面表现
author: admin
type: post
date: 2010-11-10T08:28:25+00:00
url: /archives/6601
IM_contentdowned:
 - 1
categories:
 - 前端设计

---
**为什么要从时间维度对页面进行监测？**

[网站分析](http://bluewhale.cc/ "网站分析") 中有一个重要的指标：Bounce rate。定义是当用户在进入你网站的第一个页面后没有点击任何链接就离开了。假设两个用户同时进入网站的第一个页面后，一个什么都没有看就马上离开了，而另一个在阅读了页面上的内容并在Action按钮上忧郁了半天，最后离开了。那么这两个用户在Bounce rate里看起来没有区别。而实际上第二个用户比第一个用户更有可能被转化。而如果两个用户来自不同的渠道，那么第二个渠道相对第一个渠道更有一些价值。

在GA的报告中目前只提供两个时间维度：平均 [页面停留时间](http://bluewhale.cc/2010-01-24/google-analytics-metrics-timeonpage-timeonsite.html "页面停留时间") 和平均 [网站停留时间](http://bluewhale.cc/2010-01-24/google-analytics-metrics-timeonpage-timeonsite.html "网站停留时间")。使用JS事件与_trackPageview函数相配合，我们可以看到网页在时间维度上的表现，并获得更多详细的用户与网站互动的时间数据（被计算在Bounce rate内的非点击行为数据）。

## **1通过持续时间判断用户行为的有效性**

[上一篇文章][1]的最后一个策略中提到，可以使用_trackPageview函数与JS的onmouseover事件配合。当用户将鼠标移到某个焦点图或按钮上时进行记录。但有一个问题就是收集到的大部分数据可能是用户无意识的行为（如鼠标划过或无意识的将鼠标停放在某个图标上），这并不能表明用户对这个焦点图或action按钮感兴趣。所以单纯记录这个行为并没有太大意义。

现在我们对这个行为加一个时间的限制。即只有用户在某个我们希望他关注的图片或action按钮上停留足够长的时间后再进行记录。（这样并不能百分百的解决之前的问题，但会大幅度的降低对无意义数据的记录，使数据更有参考性。）

具体操作是我们先编写一段时间控制脚本，然后当条件满足后通过onmouseover事件触发_trackPageview函数记录并发送数据。

时间控制JS代码：

1


2


3


4


5


6


7


8


9


10


1


![](”http://www.bluewhale.cc/image/twitter.jpg”)

当用户将鼠标停留在twitter.jpg图片上5秒时，\_trackPageview函数记录并发送一次mouse\_on_img数据。

## **2监测页面的加载时间**

使用JS的onload事件和_trackPageview函数监测页面加载时间的方法在[上一篇文章][1]中已经详细介绍过了。这里说一下对收集到数据的分析。

通过GA的数据透视表功能，可以对页面加载时间按网络接入方式和所在地区两个维度进行细分。详细了解某一个地区的某种网络接入方式的页面显示速度。

[![数据透视表报告](http://blog.haohtml.com/wp-content/uploads/2010/11/pivot.jpg)][2]

## **3页面加载后持续时间监测**

页面加载后持续时间的监测和通过持续时间判断用户行为的有效性的方法大致相同。编写一段时间控制JS代码，当onload事件被触发一段时间后通过_trackPageview函数记录并发送一个数据。

时间控制JS代码：

1


当页面完成加载10秒后，_trackPageview函数记录并发送一次loaded10seconds数据。


## **4用户页面停留时间监测**

用户页面停留时间在上一篇文章中已有详细介绍。请点击 [这里](http://bluewhale.cc/2010-01-12/google-analytics-trackpageview-policy.html) 查看。


## **5通过时间维度监测用户的行为**

通过时间维度的监测，我们还可以知道用户在进行某个行为前的时间。如用户在打开landing page后到点击Action按钮之间的时间。如果这个时间过长有可能是我们的Action按钮过于隐蔽，又或者是页面文字的激励效果不够好。这时可能就需要对页面的文字表述或Action按钮的位置及大小进行调整了。


最后，一个很有意思的情况是我们虽然通过对时间维度的监测又对Bounce rate里的数据有了更多的了解，但这也完全破坏了原有的Bounce rate数据。所以，还是要根据目标和实际情况进行取舍才能获得更有价值的数据。


——【所有文章及图片版权归 蓝鲸（王彦平）所有。欢迎转载，但请注明转自“ [蓝鲸网站分析博客](http://bluewhale.cc/)”。】——


 [1]: http://bluewhale.cc/2010-01-12/google-analytics-trackpageview-policy.html
 [2]: http://blog.haohtml.com/wp-content/uploads/2010/11/pivot.jpg