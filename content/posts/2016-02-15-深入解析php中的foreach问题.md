---
title: 深入解析php中的foreach问题
author: admin
type: post
date: 2016-02-15T04:53:18+00:00
url: /archives/16592
categories:
 - 程序开发
tags:
 - php

---

篇文章是对php中的foreach问题进行了详细的分析介绍，需要的朋友参考下


**前言：** php4中引入了foreach结构，这是一种遍历数组的简单方式。相比传统的for循环，foreach能够更加便捷的获取键值对。在php5之前，foreach仅能用于数组；php5之后，利用foreach还能遍历对象（详见：遍历对象）。本文中仅讨论遍历数组的情况。foreach虽然简单，不过它可能会出现一些意外的行为，特别是代码涉及引用的情况下。

下面列举了几种case，有助于我们进一步认清foreach的本质。

**问题1：**

```
$arr = array(1,2,3);
foreach($arr as $k => &$v) {
$v = $v * 2;
}
// now $arr is array(2, 4, 6)
foreach($arr as $k => $v) {
echo "$k", " => ", "$v";
}
```

先从简单的开始，如果我们尝试运行上述代码，就会发现最后输出为0=>2  1=>4  2=>4 。

为何不是0=>2  1=>4  2=>6 ？

其实，我们可以认为 foreach($arr as $k => $v) 结构隐含了如下操作，分别将数组当前的’键’和当前的’值’赋给变量$k和$v。具体展开形如：


```
foreach($arr as $k => $v){
//在用户代码执行之前隐含了2个赋值操作
$v = currentVal();
$k = currentKey();
//继续运行用户代码
……
}
```

根据上述理论，现在我们重新来分析下第一个foreach：

第1遍循环，由于$v是一个引用，因此$v = &$arr[0]，$v=$v*2相当于$arr[0]*2，因此$arr变成2,2,3

第2遍循环，$v = &$arr[1]，$arr变成2,4,3

第3遍循环，$v = &$arr[2]，$arr变成2,4,6

**随后代码进入了第二个foreach：** 第1遍循环，隐含操作$v=$arr[0]被触发，由于此时$v仍然是$arr[2]的引用，即相当于$arr[2]=$arr[0]，$arr变成2,4,2

第2遍循环，$v=$arr[1]，即$arr[2]=$arr[1]，$arr变成2,4,4

第3遍循环，$v=$arr[2]，即$arr[2]=$arr[2]，$arr变成2,4,4

OK，分析完毕。

**如何解决类似问题呢？php手册( [http://php.net/manual/zh/control-structures.foreach.php](http://php.net/manual/zh/control-structures.foreach.php))上有一段提醒：** Warning : 数组最后一个元素的 $value 引用在 foreach 循环之后仍会保留。建议使用unset()来将其销毁。

```
$arr = array(1,2,3);
foreach($arr as $k => &$v) {
$v = $v * 2;
}
unset($v);
foreach($arr as $k => $v) {
echo "$k", " => ", "$v";
}
// 输出 0=>2  1=>4  2=>6
```

从这个问题中我们可以看出，引用很有可能会伴随副作用。如果不希望无意识的修改导致数组内容变更，最好及时unset掉这些引用。


官方文档： [http://php.net/manual/zh/control-structures.foreach.php](http://php.net/manual/zh/control-structures.foreach.php)