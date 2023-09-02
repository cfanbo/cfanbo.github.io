---
title: IE下onpropertychange与firefox下oninput用法
author: admin
type: post
date: 2010-11-02T16:29:58+00:00
url: /archives/6518
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - js
 - onpropertychange

---

onpropertychange能够捕获每次输入值的变化。例如：对象的value值被改变时，onpropertychange能够捕获每次改变，而onchange需要执行某个事件才可以捕获。


onpropertychange 不被firefox所支持，如果想在firefox下正常使用,需要用oninput属性,且需要用addEventListener来注册事件。

例子：


oninput测试