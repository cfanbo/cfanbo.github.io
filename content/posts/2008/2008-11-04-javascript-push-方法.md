---
title: JavaScript push() 方法
author: admin
type: post
date: 2008-11-04T04:36:47+00:00
excerpt: |
 定义和用法
 push() 方法可向数组的末尾添加一个或多个元素，并返回新的长度。

 语法
 arrayObject.push(newelement1,newelement2,....,newelementX)
 参数 描述
 newelement1 必需。要添加到数组的第一个元素。
 newelement2 可选。要添加到数组的第二个元素。
 newelementX 可选。可添加多个元素。

 返回值
 把指定的值添加到数组后的新长度。

 说明
 push() 方法可把它的参数顺序添加到 arrayObject 的尾部。它直接修改 arrayObject，而不是创建一个新的数组。push() 方法和 pop() 方法使用数组提供的先进后出栈的功能。
url: /archives/515
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - javascript
 - push

---

## 定义和用法

push() 方法可向数组的末尾添加一个或多个元素，并返回新的长度。


### 语法

```
arrayObject.push(newelement1,newelement2,....,newelementX)
```

参数

描述

newelement1

必需。要添加到数组的第一个元素。

newelement2

可选。要添加到数组的第二个元素。

newelementX

可选。可添加多个元素。


### 返回值

把指定的值添加到数组后的新长度。


### 说明

push() 方法可把它的参数顺序添加到 arrayObject 的尾部。它直接修改 arrayObject，而不是创建一个新的数组。push() 方法和 pop() 方法使用数组提供的先进后出栈的功能。


## 提示和注释

注释：该方法会改变数组的长度。


提示：要想数组的开头添加一个或多个元素，请使用 unshift() 方法。


## 实例

在本例中，我们将创建一个数组，并通过添加一个元素来改变其长度：


```
<script type="text/javascript">

var arr = new Array(3)
arr[0] = "George"
arr[1] = "John"
arr[2] = "Thomas"

document.write(arr + "<br />")
document.write(arr.push("James") + "<br />")
document.write(arr)

</script>
```

输出：


```
George,John,Thomas
4
George,John,Thomas,James
```

## TIY

[push()](http://blog.haohtml.com/tiy/t.asp?f=jseg_push)
如何使用 push() 来改变数组的长度。