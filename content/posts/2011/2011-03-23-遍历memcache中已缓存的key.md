---
title: 遍历memcache中已缓存的key
author: admin
type: post
date: 2011-03-23T08:18:46+00:00
url: /archives/8111
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - memcache

---

最近需要做一个缓存管理的功能。其中有一个需要模糊匹配memcached的key然后进行删除匹配上的key对应的数据。


难点在于memcache 没有提供模糊匹配key删除缓存的功能，也没有提供遍历memcache key的功能。在网上search了下，


点击10个连接有9个都是一篇文章的copy。网上最流行的方法发现key不能正确的获得。baidu google 全用上了就是没有解决。。。


最后还是抱着试试的心态，终于把问题给解决了。废话少说。说说，我实现的代码：


遍历memcache的可以需要有一下几个步骤：


1、通过使用memcache 内置方法Memcache::getExtendedStats，首先获得items信息。


最后得到的解决类似与

```

  1 $memcache = new Memcache();
  2
  3  $all_items = $memcache->getExtendedStats('items');
  4
  5 var_export($all_items);

```

```

  1 array (
   2   '192.168.0.110:11211' =>
   3   array (
   4     'items' =>
   5     array (
   6       1 =>
   7       array (
   8         'number' => '1',
   9         'age' => '1851',
  10       ),
  11       2 =>
  12       array (
  13         'number' => '1',
  14         'age' => '1851',
  15       ),
  16       3 =>
  17       array (
  18         'number' => '2',
  19         'age' => '1864',
  20       ),
  21       7 =>
  22       array (
  23         'number' => '1',
  24         'age' => '1851',
  25       ),
  26       9 =>
  27       array (
  28         'number' => '1',
  29         'age' => '1',
  30       ),
  31       12 =>
  32       array (
  33         'number' => '2',
  34         'age' => '1851',
  35       ),
  36       13 =>
  37       array (
  38         'number' => '1',
  39         'age' => '1851',
  40       ),
  41       14 =>
  42       array (
  43         'number' => '1',
  44         'age' => '1851',
  45       ),
  46       15 =>
  47       array (
  48         'number' => '1',
  49         'age' => '1851',
  50       ),
  51       16 =>
  52       array (
  53         'number' => '1',
  54         'age' => '1850',
  55       ),
  56       18 =>
  57       array (
  58         'number' => '2',
  59         'age' => '1851',
  60       ),
  61       19 =>
  62       array (
  63         'number' => '1',
  64         'age' => '1851',
  65       ),
  66       20 =>
  67       array (
  68         'number' => '1',
  69         'age' => '1851',
  70       ),
  71     ),
  72   ),
  73 )

```

$all_items中的key“192.168.0.110:11211” 就是memcache的host和port。


2、已$all_items做为数据源，再次调用Memcache::getExtendedStats，我们需要的数据就在返回的结果里面


﻿我们假设memcache所有的host信息为$options = array(‘192.168.0.110:11211’,);


﻿


```

  1 foreach ($options as $option) {
   2      if (isset($all_items[$option]['items'])) {
   3            $items      = $all_items[$option]['items'];
   4
   5            foreach ($items as $number => $item) {
   6                  $str    = $memcache->getExtendedStats('cachedump', $number, 0);
   7                  $line   = $str[$option];
   8                  if (is_array($line) && count($line) > 0) {
   9                       foreach ($line as $key => $value) {
  10                            $keys[] = $key;
  11                       }
  12                  }
  13            }
  14       }
  15 }

```

上面的$keys数组就是我们需要的数据了。


﻿下面贴出来完整的代码


﻿


```

  1 function list_key() {
   2     $memcache = new Memcache();
   3     $all_items = $memcache->getExtendedStats('items');
   4     $keys      = array();
   5     foreach ($this->_options as $options) {
   6         foreach ($options as $option) {
   8         　　if (isset($all_items[$option]['items'])) {
   9                  $items = $all_items[$option]['items'];
  10                  foreach ($items as $number => $item) {
  11                        $str    = $memcache->getExtendedStats('cachedump', $number, 0);
  12                        $line   = $str[$option];
  13                    　　if (is_array($line) && count($line) > 0){

  　　　　　　　　　　　　　　　　foreach ($line as $key => $value) {

  $keys[] = $key;
  14                      　　 }
  15                    　　}
  16                }
  17            }
  18         }
  19     }
  20
  21     return array_unique($keys);
  22
  23 }

```

﻿