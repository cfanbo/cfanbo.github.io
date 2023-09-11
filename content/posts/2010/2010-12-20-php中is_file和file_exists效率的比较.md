---
title: php中is_file和file_exists效率的比较
author: admin
type: post
date: 2010-12-20T02:05:36+00:00
url: /archives/7063
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - php

---
下面是测试代码,分别循环10000次：

>

>

>
>

> $start_time = get_microtime();
>

>
>

> for($i=0;$i<10000;$i++)
>

>
>

> {
>

>
>

> if(is_file(‘url.txt’)) {
>

>
>

> //do nothing;
>

>
>

> }
>

>
>

> }
>

>
>

> echo ‘is_file耗时–>’.(get_microtime() – $start_time).'

’;
>

>
>

> $start_time = get_microtime();
>

>
>

> for($i=0;$i<10000;$i++)
>

>
>

> {
>

>
>

> if(file_exists(‘url.txt’)) {
>

>
>

> //do nothing;
>

>
>

> }
>

>
>

> }
>

>
>

> echo ‘file_exits–>’.(get_microtime() – $start_time).'

’;
>

>
>

> function get_microtime()//时间
>

>
>

> {
>

>
>

> list($usec, $sec) = explode(‘ ‘, microtime());
>

>
>

> return ((float)$usec + (float)$sec);
>

>
>

> }
>

>
>

> ?>
>

**文件不存在时，结果为：**

is_file耗时–>12.006669998169

file_exits–>7.0664341449738**文件存在时，结果为：**is_file耗时–>4.6614780426025

 file_exits–>6.3218870162964

**结论：**

****看起来file_exits函数并不会因为该文件是否真的存在而影响速度，但是is_file影响就大了。

原因可能是，file\_exists 就是is\_file和is_dir的结合体啊,一个人干两个人的活当然慢了.