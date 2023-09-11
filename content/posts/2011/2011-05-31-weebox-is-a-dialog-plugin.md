---
title: weebox is a dialog plugin
author: admin
type: post
date: 2011-05-30T23:24:27+00:00
url: /archives/9625
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - jQuery
 - weebox

---
使用前需包含以下jquery.js、bgiframe.js、weebox.js、wee.css文件

基本用法举例如下:
$.weeboxs.open(‘#testbox’, {title: ‘hello world’, width:400, height: 200});

$.weeboxs.open(‘The operation failed.’,{
onopen:function(){alert(‘opened!’);},
onclose:function(){alert(‘closed!’);}, onok:function(){alert(‘ok’);
$.weeboxs.close();} });

$.weeboxs.open(‘/modules/test/testsession.php’, {contentType:’ajax’});

$.weeboxs.open(‘hello world’);

$.weeboxs.open(‘The operation failed.’,{type:’error’});

$.weeboxs.open(‘The operation failed.’,{type:’wee’});

$.weeboxs.open(‘The operation failed.’,{type:’success’});

$.weeboxs.open(‘The operation failed.’,{type:’warning’});

$.weeboxs.open(‘Autoclosing in 5 seconds.’, { timeout: 5 });

**以下是默认选项:**

boxid: null, //设定了此值只后，以后在打开同样boxid的弹窗时，前一个将被自
动关闭

boxclass: null, //给弹窗设置其它的样式，用此可以改变弹窗的样式

type: ‘dialog’, //弹窗类型，目前有dialog,error,warning,success,wee,prompt,
box六种

title: ”, //弹窗标题

width: 0, //弹窗宽度,不设时，会自动依据内容改变大小

height: 0, //弹窗高度(注意是内容的高度,不是弹窗的高度)

timeout: 0, //自动关闭的秒数，设置此值后，窗口将自动关闭

draggable: true,//是否可以拖拽

modal: true, //是否显示遮照

overlay: 75, //遮照透明度

focus: null, //弹窗打开后，焦点移到什么元素上，默认移到取消按钮到

position: ‘center’,//弹窗打开后的默认为中间，设置为element时，需要设置trager选项，

trigger: null, //显示位置的参照元素，为一个元素id

showTitle: true,//是否显示标题

showButton: true,//是否显示按钮，包括确定和取消

showCancel: true, //是否显示取消按钮

showOk: true, //是否显示确定按钮

okBtnName: ‘确定’,//”确定”按钮名称

cancelBtnName: ‘取消’,//”取消”按钮名称

contentType: ‘text’,//内容获取方式，目前有三种text,selector,ajax

contentChange: false,//为selector时

clickClose: false, //点击不在弹窗上时，是否关闭弹窗

zIndex: 999,//默认弹窗的层

animate: false,//效果显示

onclose: null, //弹窗关闭时触发的函数

onopen: null, //弹窗显示前触发的函数, 此时内容已经放入弹窗中，不过还没有显
示出来

onok: null //点击确定按钮触发的函数



官方地址: [http://plugins.jquery.com/node/4473](http://plugins.jquery.com/node/4473)

参考: [http://code.google.com/p/weebox/](http://code.google.com/p/weebox/)