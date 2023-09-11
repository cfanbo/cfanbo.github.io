---
title: attachEvent与addEventListener区别
author: admin
type: post
date: 2009-05-21T03:30:54+00:00
excerpt: |
 适应的浏览器版本不同，同时在使用的过程中要注意
 attachEvent方法 按钮onclick
 addEventListener方法 按钮click

 两者使用的原理：可对执行的优先级不一样，下面实例讲解如下：
 attachEvent方法，为某一事件附加其它的处理事件。（不支持Mozilla系列）

 addEventListener方法 用于 Mozilla系列
url: /archives/1426
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - javascript

---
适应的浏览器版本不同，同时在使用的过程中要注意
attachEvent方法          按钮onclick
addEventListener方法    按钮click

有关addEventListener函数的相关参数 [请点击这里查看](/index.php/archives/1429)

两者使用的原理：可对执行的优先级不一样，下面实例讲解如下：
attachEvent方法，为某一事件附加其它的处理事件。（不支持Mozilla系列）

addEventListener方法 用于 Mozilla系列

举例: document.getElementById(“btn”).onclick = method1;
document.getElementById(“btn”).onclick = method2;
document.getElementById(“btn”).onclick = method3;如果这样写,那么将会只有medhot3被执行

写成这样：
var btn1Obj = document.getElementById(“btn1”); //object.attachEvent(event,function);
btn1Obj.attachEvent(“onclick”,method1);
btn1Obj.attachEvent(“onclick”,method2);
btn1Obj.attachEvent(“onclick”,method3);执行顺序为method3->method2->method1

如果是Mozilla系列，并不支持该方法，需要用到addEventListener var btn1Obj = document.getElementById(“btn1”);
//element.addEventListener(type,listener,useCapture);
btn1Obj.addEventListener(“click”,method1,false);
btn1Obj.addEventListener(“click”,method2,false);
btn1Obj.addEventListener(“click”,method3,false);执行顺序为method1->method2->method3

使用实例：

1。 var el = EDITFORM_DOCUMENT.body;
//先取得对象，EDITFORM_DOCUMENT实为一个iframe
if (el.addEventListener){
el.addEventListener(‘click’, KindDisableMenu, false);
} else if (el.attachEvent){
el.attachEvent(‘onclick’, KindDisableMenu);
}2。 if (window.addEventListener) {
window.addEventListener(‘load’, _uCO, false);
} else if (window.attachEvent) {
window.attachEvent(‘onload’, _uCO);
}