---
title: JavaScript isNaN() 函数
author: admin
type: post
date: 2008-11-04T04:46:33+00:00
excerpt: |
 定义和用法
 isNaN() 函数用于检查其参数是否是非数字值

 语法
 isNaN(x)
 参数 描述
 x 必需。要检测的值。
 返回值
 如果 x 是特殊的非数字值 NaN（或者能被转换为这样的值），返回的值就是 true。如果 x 是其他值,则返回 false。
 说明
 isNaN() 函数可用于判断其参数是否是 NaN，该值表示一个非法的数字（比如被 0 除后得到的结果）。
 如果把 NaN 与任何值（包括其自身）相比得到的结果均是 false，所以要判断某个值是否是 NaN，不能使用 == 或 === 运算符。正因为如此，isNaN() 函数是必需的。
url: /archives/518
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - javascript

---

## 义和用法

isNaN() 函数用于检查其参数是否是非数字值。


### 语法

```
isNaN(x)
```

 参数

 描述

 x

 必需。要检测的值。


### 返回值

如果 x 是特殊的非数字值 NaN（或者能被转换为这样的值），返回的值就是 true。如果 x 是其他值,则返回 false。


### 说明

isNaN() 函数可用于判断其参数是否是 NaN，该值表示一个非法的数字（比如被 0 除后得到的结果）。


如果把 NaN 与任何值（包括其自身）相比得到的结果均是 false，所以要判断某个值是否是 NaN，不能使用 == 或 === 运算符。正因为如此，isNaN() 函数是必需的。


## 提示和注释

提示：isNaN() 函数通常用于检测 parseFloat() 和 parseInt() 的结果，以判断它们表示的是否是合法的数字。当然也可以用 isNaN() 函数来检测算数错误，比如用 0 作除数的情况。


## 实例

在本例中，我们将使用 isFinite() 在检测无穷数：


```
<script type="text/javascript">

document.write(isFinite(123))	//返回 false
document.write(isFinite(-1.23))	//返回 false
document.write(isFinite(5-2))	//返回 false
document.write(isFinite(0))		//返回 false
document.write(isFinite(0/0))	//返回 true
document.write(isFinite("Hello"))	//返回 true
document.write(isFinite("2005/12/12"))	//返回 true
document.write(isFinite(true))	//返回 false
document.write(isFinite(undefined))	//返回 true

</script>
```

## TIY

[isNaN()](http://blog.haohtml.com/tiy/t.asp?f=jseg_isNaN)
 如何使用 isNaN()。