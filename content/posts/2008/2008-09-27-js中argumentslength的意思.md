---
title: js中arguments.length的意思
author: admin
type: post
date: 2008-09-27T07:40:51+00:00
url: /archives/423
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - javascript

---
function imagePreload() {
var imgPreload = new Image();
for (i = 0; i < arguments.length; i++) {
imgPreload.src = arguments[i];
}
}
**imagePreload(‘001.gif’, ‘002.gif’, ‘003.gif’, ‘004.gif’, ‘005.gif’)**

这个是js中的arguments.主要是可以对输入的参数进行跟踪。
这如作者所举出的例子:imagePreload函数出入了5个参数，所以在js代码中的
arguments.length会知道你输入的了5个参数。并可以通过索引器获得五个参数的值。