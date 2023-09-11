---
title: mootools基本XMLHttpRequest的包装类
author: admin
type: post
date: 2008-03-01T12:55:36+00:00
url: /archives/274
IM_contentdowned:
 - 1
categories:
 - 前端设计

---

# [top](http://www.seoeye.org/mootools/XHR.htm\#MainTopic) XHR.js

包含了基本的 XMLHttpRequest 类的包装〿


#### License

MIT-style license.


概要


[XHR.js](http://www.seoeye.org/mootools/XHR.htm#XHR.js)
包含了基本的 XMLHttpRequest 类的包装〿
[XHR](http://www.seoeye.org/mootools/XHR.htm#XHR)
基本皿XMLHttpRequest的包装类
[属怿/a>](http://www.seoeye.org/mootools/XHR.htm#XHR.Properties)[setHeader](http://www.seoeye.org/mootools/XHR.htm#XHR.setHeader)
添加/修改请求的Header
[send](http://www.seoeye.org/mootools/XHR.htm#XHR.send)
打开XMLHttpRequest连接并发送数捿/td>
[cancel](http://www.seoeye.org/mootools/XHR.htm#XHR.cancel)
取消正在执行的请汿


## [top](http://www.seoeye.org/mootools/XHR.htm\#MainTopic) XHR

基本皿XMLHttpRequest的包装类


#### 参数

options

一个请求的配置对象。参考下面的可选项


#### 可选项

method

’post’ 房‘get’ – 请求的协访 可选，默认丿lsquo;post’.

async

是否是异步。默认为true.

encoding

数据编码。默认为utf-8.

autoCancel

自动取消前一个正在执行的请求。默认为false.

headers

一个请求头的配置对豿


#### 事件

onRequest

请求发送时触发

onSuccess

请求完成时触叿

onStateChange

XMLHttpRequest状态发生改变时触发

onFailure

XMLHttpRequest状态为失败时触叿/td>


#### 属怿

running

请求是否正在执行

response

请求的返回对象。对象中包含的键有text和xml。可以在onSuccess事件中访问到这个对象〿


#### 示例

> ```
> var myXHR = new XHR({method:'get'}).send('http://site.com/requestHandler.php','name=john&lastname=dorian');
>
> ```

概要


[属怿/a>](http://www.seoeye.org/mootools/XHR.htm#XHR.Properties)[setHeader](http://www.seoeye.org/mootools/XHR.htm#XHR.setHeader)
添加/修改请求的Header
[send](http://www.seoeye.org/mootools/XHR.htm#XHR.send)
打开XMLHttpRequest连接并发送数捿
[cancel](http://www.seoeye.org/mootools/XHR.htm#XHR.cancel)
取消正在执行的请汿


### [top](http://www.seoeye.org/mootools/XHR.htm\#MainTopic) Properties

### [top](http://www.seoeye.org/mootools/XHR.htm\#MainTopic) setHeader

添加/修改请求的Header。它不会覆盖在可选项中指定的Header〿


#### 示例

> ```
> var myXhr = new XHR(url, {method: 'get', headers: {'X-Request':'JSON'}});
> myXhr.setHeader('Last-Modified','Sat, 1 Jan 2005 05:00:00 GMT');
>
> ```

### [top](http://www.seoeye.org/mootools/XHR.htm\#MainTopic) send

打开XMLHttpRequest连接并发送数据。数据可以是null或者是字符丿


#### 示例

> ```
> var myXhr = new XHR({method: 'post'});
> myXhr.send(url, querystring);
> var syncXhr = new XHR({async: false, method: 'post'});
> syncXhr.send(url, null);
>
> ```

### [top](http://www.seoeye.org/mootools/XHR.htm\#MainTopic) cancel

取消正在执行的请求。如果请求不在执行，则不会发生作用〿


#### 示例

> ```
> var myXhr = new XHR({method: 'get'}).send(url);
> myXhr.cancel();
>
> ```