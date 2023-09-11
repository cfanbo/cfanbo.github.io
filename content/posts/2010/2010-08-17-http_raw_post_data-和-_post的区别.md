---
title: $HTTP_RAW_POST_DATA 和 $_POST的区别
author: admin
type: post
date: 2010-08-17T01:01:57+00:00
url: /archives/5130
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php

---
这是手册里写的

总是产生变量包含有原始的 POST 数据。否则，此变量仅在碰到未识别 MIME 类型的数据时产生。不过，访问原始 POST 数据的更好方法是 php://input。$HTTP\_RAW\_POST_DATA 对于 enctype=”multipart/form-data” 表单数据不可用。

问题：    $HTTP\_RAW\_POST\_DATA  == $\_POST  吗？

照手册所写 ，答案应该就为否。
假如不一样的话，他们的区别是什么呢？

我知道答案了，如下：

The RAW / uninterpreted HTTP POst information can be accessed with:
$GLOBALS[‘HTTP\_RAW\_POST_DATA’]

This is useful in cases where the post Content-Type is not something PHP understands (such as text/xml).

也就是说,基本上$GLOBALS[‘HTTP\_RAW\_POST\_DATA’] 和 $\_POST是一样的。但是如果post过来的数据不是PHP能够识别的，你可以用 $GLOBALS[‘HTTP\_RAW\_POST_DATA’]来接收，比如 text/xml 或者 soap 等等。

PHP默认识别的数据类型是application/x-www.form-urlencoded标准的数据类型。