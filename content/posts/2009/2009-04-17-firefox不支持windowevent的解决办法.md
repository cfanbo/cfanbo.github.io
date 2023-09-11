---
title: firefox不支持window.event的解决办法
author: admin
type: post
date: 2009-04-16T23:00:50+00:00
url: /archives/1240
IM_contentdowned:
 - 1
categories:
 - 前端设计

---
在最前面的javascript加上以下语句:

1. //首先，定义一个全局的event
2. if( typeof(window.event)==“undefined” ){
3. eval(“var event = new Object;”);
4. }