---
title: php中$_request与$_post、$_get的区别
author: admin
type: post
date: 2008-11-21T00:37:45+00:00
excerpt: |
 |
 php中有$_request与$_post、$_get用于接受表单数据，当时他们有何种区别，什么时候用那种最好。

 一、$_request与$_post、$_get的区别和特点

 $_REQUEST[]具用$_POST[] $_GET[]的功能,但是$_REQUEST[]比较慢。通过post和get方法提交的所有数据都可以通过$_REQUEST数组获得

 二、$_post、$_get的区别和特点

 1. get是从服务器上获取数据，post是向服务器传送数据。
 2. get是把参数数据队列加到提交表单的ACTION属性所指的URL中，值和表单内各个字段一一对应，在URL中可以看到。post是通过HTTP post机制，将表单内各个字段与其内容放置在HTML HEADER内一起传送到ACTION属性所指的URL地址。用户看不到这个过程。
url: /archives/625
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php

---
php中有$\_request与$\_post、$_get用于接受表单数据，当时他们有何种区别，什么时候用那种最好。

一、$\_request与$\_post、$_get的区别和特点

$\_REQUEST[]具用$\_POST[] $\_GET[]的功能,但是$\_REQUEST[]比较慢。通过post和get方法提交的所有数据都可以通过$_REQUEST数组获得

二、$\_post、$\_get的区别和特点

1. get是从服务器上获取数据，post是向服务器传送数据。
2. get是把参数数据队列加到提交表单的ACTION属性所指的URL中，值和表单内各个字段一一对应，在URL中可以看到。post是通过HTTP post机制，将表单内各个字段与其内容放置在HTML HEADER内一起传送到ACTION属性所指的URL地址。用户看不到这个过程。
3. 对于get方式，服务器端用Request.QueryString获取变量的值，对于post方式，服务器端用Request.Form获取提交的数据。
4. get传送的数据量较小，不能大于2KB。post传送的数据量较大，一般被默认为不受限制。但理论上，IIS4中最大量为80KB，IIS5中为100KB。
5. get安全性非常低，post安全性较高。

举例：mypage?id=1这种就是GET方式传值，可以用$\_request和$\_get接受传值。

例2：HTTP请求有POST和GET。在写表单form时可以指定action为post或get。数组$\_POST中保存POST方法传递的变量， $\_GET保存GET方法传递的变量。$_REQUEST中包含二者。
例如

在t.php中，可以使用$_GET[‘aaa’]获得网页表单中填写的数据.

当form中的action为get时使用$\_GET；action为post时用$\_POST。二者都可用 $_REQUEST