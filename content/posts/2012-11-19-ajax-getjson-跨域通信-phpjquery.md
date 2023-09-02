---
title: Ajax getjson 跨域通信 php+jquery
author: admin
type: post
date: 2012-11-19T08:50:00+00:00
url: /archives/13501
categories:
 - 程序开发
tags:
 - 跨域
 - getjson
 - jQuery

---
**网站A的表单提交部分：**

**网站B的输出json部分：**

$_GET[‘jsoncallback’] . ‘(‘ . json_encode($return) . ‘)‘);

?>