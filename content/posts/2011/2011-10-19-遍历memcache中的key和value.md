---
title: 遍历memcache中的key和value
author: admin
type: post
date: 2011-10-19T05:51:01+00:00
url: /archives/11795
IM_contentdowned:
 - 1
categories:
 - nosql
tags:
 - memcache

---

**什么是** **memcache**

memcache是一个高性能的分布式的内存对象缓存系统，通过在内存里维护一个统一的巨大的hash表，它能够用来存储各种格式的数据，包括图像、视频、文件以及数据库检索的结果等。Memcache是danga.com的一个项目，最早是为 LiveJournal 服务的，最初为了加速 LiveJournal 访问速度而开发的，后来被很多大型的网站采用。目前全世界不少人使用这个缓存项目来构建自己大负载的网站，来分担数据库的压力。


**为什么要遍历**

目前，用到memcache的公司和网站也越来越多。Memcache的客户端操作一般都只提供了get,set等简单的操作，这些操作都是非常高效的。   虽然memcache是个key-value存储的系统，但是在某些时候，我们可能需要遍历memcache的数据。

通过使用memcache 内置方法Memcache::getExtendedStats，首先获得items信息。


最后得到的解决类似与


```
<php
$memcache = new Memcache();
$all_items = $memcache->getExtendedStats('items');
var_export($all_items);
?>
```

其中$all_items中的key“192.168.0.110:11211” 就是memcache的host和port。

**如何遍历** **memcache**

**stats** **命令**

memcache的stats命令包括：


1.        stats


2.        stats reset


3.        stats malloc


4.        stats maps


5.        stats sizes


6.        stats slabs


7.        stats items


8.        stats cachedump slab_id limit_num


9.        stats detail [on|off|dump]


**通过命令完成遍历**

通过这些stats命令我们就可以完成memcache存储的内容的遍历，OK，下面我们通过telnet直接连接到memcache通过这些命令来完成相关的操作。

telnet到192.168.15.225（局域网测试机器）的memcache服务器

执行stats items命令，可以看到出现 很多的items行。


执行stats cachedump 3 0命令。这里的3表示上面图中items后面的数字，0标示显示全部的数据，如果是1就标示只显示1条。

下图为执行后的结果，item后面的字符串为key

通过上面列出的key我们就可以遍历所有的数据了，下面我们取出某一条数据，key为Uc!uLh的数据。

到这里，你也许明白了怎么去遍历memcache的数据了。


**代码实现**

下面贴上一段php实现的遍历memcache数据的代码，其他语言可以参考代码自己实现。


```
<?php

$host='225';
$port=11211;

$mem=new Memcache();
$mem->connect($host,$port);

$items=$mem->getExtendedStats ('items');
$items=$items["$host:$port"]['items'];
for($i=0,$len=count($items);$i<$len;$i++){

$number=$items[$i]['number'];

 $str=$mem->getExtendedStats ("cachedump",$number,0);

 $line=$str["$host:$port"];

 if( is_array($line) && count($line)>0){

     foreach($line as $key=>$value){

         echo $key . '=>';

         print_r($mem->get($key));

         echo "\r\n";

     }

 }

}

?>
```

**扩展功能**

由此可以实现查找memcache某个前缀的key的数据，或者查询某些value的key。甚至实现数据库的like功能。请注意：遍历memcache的操作并没有memcache的get操作那么高效。