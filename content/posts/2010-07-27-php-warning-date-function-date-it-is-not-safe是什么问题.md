---
title: 'PHP Warning: date() [function.date]: It is not safe是什么问题'
author: admin
type: post
date: 2010-07-27T04:46:51+00:00
url: /archives/4837
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php

---
**在用PHP5.3以上的PHP版本时，只要是涉及时间的会报一个”** PHP Warning: date() [function.date]: It is not safe to rely on the system’s timezone settings. You are *required* to use the date.timezone setting or the date_default_timezone_set() function. In case you used any of those methods and you are still getting this warning, you most likely misspelled the timezone identifier. We selected ‘UTC’ for ‘8.0/no DST’ instead in **“这样的错。如何解决呢？****实际上，从 PHP 5.1.0 ，当对使用date（）等函数时，如果timezone设置不正确，在每一次调用时间函数时,都会产生`E_NOTICE` 或者 `E_WARNING` 信息。而又在php5.1.0 中，date.timezone这个选项，默认情况下是关闭的，无论用什么php命令都是格林威治标准时间，但是** PHP5.3 中好像如果没有设置也会强行抛出了这个错误的,解决此问题，只要本地化一下，就行了。**以下 是两种方法(任选一种都 行)：**

****

一、在页头使用 date_default_timezone_set()设置date_default_timezone_set(‘PRC’); //东八时区

 echo date(‘Y-m-d H:i:s’);二、修改php.ini。

 打开php5.ini查找date.timezone 去掉前面的分号　= 后面加XXX，重启http服务（如apache2或iis等）即可。XXX可以任意正确的值。对于我们国内来 说：可以为以下值：Asia/Chongqing ，Asia/Shanghai ，Asia/Urumqi （依次为重庆，上海，乌鲁木齐）港台地区可用：Asia/Macao ，Asia/Hong_Kong ，Asia/Taipei （依次为澳门，香港，台北），还有新加坡：Asia/Singapore，当然PRC也行。