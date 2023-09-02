---
title: Golang中的unsafe.Sizeof()简述
author: admin
type: post
date: 2018-10-19T10:15:25+00:00
url: /archives/18473
categories:
 - 程序开发
tags:
 - golang

---
测试环境:
系统 win7 64位
go version: go1.10 windows/amd64

我们先看一下代码的输出

```
package main

import "unsafe"

func main() {
	// string
	str1 := "abc"
	println("string1:", unsafe.Sizeof(str1)) // 16
	str2 := "abcdef"
	println("string2:", unsafe.Sizeof(str2)) // 16

	// 数组
	arr1 := [...]int{1, 2, 3, 4}
	println("array1:", unsafe.Sizeof(arr1)) // 32 = 8 * 4

	arr2 := [...]int{1, 2, 3, 4, 5}
	println("array2:", unsafe.Sizeof(arr2)) // 40 = 8 * 5

	// slice 好多人分不清切片和数组的写法区别，其实只要记住[]中间是空的就是切片，反之则是数组即可
	slice1 := []int{1, 2, 3, 4}
	println("slice1:", unsafe.Sizeof(slice1)) // 24

	slice2 := []int{1, 2, 3, 4, 5}
	println("slice2:", unsafe.Sizeof(slice2)) // 24

	slice3 := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	println("slice3:", unsafe.Sizeof(slice3)) // 24
}

```

**1、字符串类型**

为什么字符串类型的 unsafe.Sizeof() 一直是16呢？

实际上字符串类型对应一个结构体，该结构体有两个域，第一个域是指向该字符串的指针，第二个域是字符串的长度，每个域占8个字节，但是并不包含指针指向的字符串的内容，这也就是为什么sizeof始终返回的是16。

组成可以理解成此结构体

```
typedef struct {
    char* buffer;
    size_tlen;
} string;

```

**2、数组类型**

编译的时候系统自动分配内存，int的长度是由系统平台来决定的，我用的是64位的系统，所以一个int 代表的就是 int64 数据类型，每个数字占用8个字节，成员数组元素个数正好等于输出的值。byte(1字节)，uint8(1字节)，uint16(2字节)，uint32(4字节)，uint64(占用8字节), byte是uint8的别名，这些全是无符号型的，对应的还有有符号型的，区别也就是一个值范围不同而已。

不同数据类型占用字节大小如下：

```
func main() {
	var a uint8
	a = 2
	fmt.Println("uint8 type size:", unsafe.Sizeof(a))

	var b uint16
	b = 2
	fmt.Println("uint16 type size:", unsafe.Sizeof(b))


	var c uint32
	c = 2
	fmt.Println("uint32 type size:", unsafe.Sizeof(c))

	var d uint64
	d = 2
	fmt.Println("uint64 type size:", unsafe.Sizeof(d))
}

```

数据类型

具体类型取值范围int8-128到127uint80到255int16-32768到32767uint160到65535int32-2147483648到2147483647uint320到4294967295int64-9223372036854775808到9223372036854775807uint640到18446744073709551615

**3、切片类型**

可以看到切片和数组还是有些不一样的，我们看一下官方包的解释 /src/unsafe/unsafe.go

> // Sizeof takes an expression x of any type and returns the size in bytes
>
>
> // of a hypothetical variable v as if v was declared via var v = x.
>
>
> // The size does not include any memory possibly referenced by x.
>
>
> // For instance, if x is a slice, Sizeof returns the size of the slice
>
>
> // descriptor, not the size of the memory referenced by the slice.

意思是说如果是slice的话，则返回的是slice描述符的长度，而不是slice的内存长度。