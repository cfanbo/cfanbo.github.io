---
title: 'FireFox 3.5+ 已不再支持 -moz-opacity'
author: admin
type: post
date: 2009-05-21T06:22:40+00:00
excerpt: |
 安装了FireFox3.5之后，发现以前项目网页中有透明属性的一些DIV都不透明了。于是猜想，FireFox3.5难道不支持它自家的CSS透明属性-moz-opacity了？上网一查，果真如此。
 在https://developer.mozilla.org/En/CSS:-moz-opacity里说得很清楚了：
 Note: Firefox 3.5 and later do not support -moz-opacity. By now, you should be using simply opacity.
 现在都要改用opacity这个属性。
url: /archives/1435
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - css
 - firefox

---
安装了FireFox3.5之后，发现以前项目网页中有透明属性的一些DIV都不透明了。于是猜想，FireFox3.5难道不支持它自家的CSS透明属性-moz-opacity了？上网一查，果真如此。
在 [https://developer.mozilla.org/En/CSS:-moz-opacity](https://developer.mozilla.org/En/CSS:-moz-opacity) 里说得很清楚了：
Note:  Firefox 3.5 and later do not support -moz-opacity.  By now, you should be using simply opacity.
现在都要改用opacity这个属性。

于是要设置一下透明度为60%的DIV就应该这样写了：
div.transp { /\* make the div translucent \*/
opacity: 0.6;                /* Firefox, Safari(WebKit), Opera)
filter: “alpha(opacity=60)”; /\* IE 8 \*/
filter: alpha(opacity=60);   /\* IE 4-7 \*/
zoom: 1;                     /\* needed in IE up to version 7, or set width or height to trigger “hasLayout” \*/
}

opacity这个是属于CSS3里面的东西了，属于CSS3的标准。然而微软IE8还不支持这一属性。

网易邮箱目前也是用-moz-opacity，因此当“清空垃圾邮件”时屏幕一片黑。