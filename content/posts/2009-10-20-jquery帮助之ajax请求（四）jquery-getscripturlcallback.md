---
title: 'jQuery帮助之Ajax请求（四）jQuery.getScript(url,[callback])'
author: admin
type: post
date: 2009-10-19T20:33:59+00:00
excerpt: '之前对jQuery.getJSON(url,[data],[callback])有一定的了解了，现在了解一下通过 HTTP GET 请求载入并执行一个 JavaScript 文件的jQuery.getScript(url,[callback])。'
url: /archives/2537
IM_data:
 - 'a:1:{s:36:"http://www.flywe.net/images/code.gif";s:64:"http://blog.haohtml.com/wp-content/uploads/2009/10/8685_code.gif";}'
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - jQuery

---
之前对jQuery.getJSON(url,[data],[callback])有一定的了解了，现在了解一下通过 HTTP GET 请求载入并执行一个 JavaScript 文件的jQuery.getScript(url,[callback])。

**jQuery.getScript(url,[callback])**

通过 HTTP GET 请求载入并执行一个 JavaScript 文件。

jQuery 1.2 版本之前，getScript 只能调用同域 JS 文件。 1.2中，您可以跨域调用 JavaScript 文件。注意：Safari 2 或更早的版本不能在全局作用域中同步执行脚本。如果通过 getScript 加入脚本，请加入延时函数。

返回值：XMLHttpRequest

参数：
**url** (String) : 待载入 JS 文件地址。
**callback** (Function) : (可选)成功载入后回调函数。

示例：
载入 [jQuery 官方颜色动画插件](http://plugins.jquery.com/project/color) 成功后绑定颜色变化动画。
**HTML 代码**：

![程序代码](http://www.flywe.net/images/code.gif) 程序代码


» Run

**jQuery代码**：

![程序代码](http://www.flywe.net/images/code.gif) 程序代码


jQuery.getScript(“http://dev.jquery.com/view/trunk/plugins/color/jquery.color.js”,

function(){

$(“#go”).click(function(){

$(“.block”).animate( { backgroundColor: ‘pink’ }, 1000)

.animate( { backgroundColor: ‘blue’ }, 1000);

});

});


加载并执行 test.js。
**jQuery代码**：

![程序代码](http://www.flywe.net/images/code.gif) 程序代码


$.getScript(“test.js”);


加载并执行 test.js ，成功后显示信息。
**jQuery代码**：

![程序代码](http://www.flywe.net/images/code.gif) 程序代码


$.getScript(“test.js”, function(){

alert(“Script loaded and executed.”);

});