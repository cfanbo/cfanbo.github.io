---
title: RESTful Web Service Cookbook 学习笔记
author: admin
type: post
date: 2013-02-18T12:35:34+00:00
url: /archives/13670
categories:
 - 系统架构
tags:
 - rest

---
每个HTTP方法都具有特定的主义.
GET　的目的是得到一个资源的表述
PUT　用于建立或更新一个资源
DELETE　用于删除一个资源
POST　用于创建多个新资源或者对资源进行多种其它变更

> 不要将GET方法用于不安全或非幂等操作.因为这样做可能会造成永久性的、不到的、不符合需要的资源改变。

在所有上述方法中，GET被滥用的情况最少，因为GET既安全又幂等。

[![crud](http://blog.haohtml.com/wp-content/uploads/2013/02/crud.png)][1]

参考：

[![rest-mi](http://blog.haohtml.com/wp-content/uploads/2013/02/rest-mi.png)][2]

 [1]: http://blog.haohtml.com/wp-content/uploads/2013/02/crud.png
 [2]: http://blog.haohtml.com/wp-content/uploads/2013/02/rest-mi.png