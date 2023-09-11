---
title: 'Runtime: 理解Golang中接口interface的底层实现'
author: admin
type: post
date: 2021-02-15T09:19:14+00:00
url: /archives/23047
categories:
 - 程序开发
tags:
 - golang
 - runtime

---
接口类型是Golang中是一种非常非常常见的`数据类型`，每个开发人员都很有必要知道它到底是如何使用的，如果了解了它的底层实现就对开发就更有帮助了。

# 接口的定义 

在Golang中 `interface` 通常是指实现了一 组抽象方法的集合，它提供了一种无侵入式的方式。当你实现了一个接口中指定的所有方法的时候，那么就实现了这个接口，在Golang中对它的实现并不需要 `implements` 关键字。

有时候我们称这种模型叫做鸭子模型（Duck typing），维基百科对鸭子模型的定义是

> ”If it looks like a duck, swims like a duck, and quacks like a duck, then it probably is a duck.“

翻译过来就是 ”如果它看起来像鸭子，像鸭子一样游泳，像鸭子一样嘎嘎叫，那他就可以认为是鸭子“。

Go 不同版本之间interface的结构可能不太一样，但整体都差不多，这里使用的Go版本为 1.15.6。

# 数据结构 

Go 中 `interface` 在运行时可分 `eface` 和 `iface` 两种数据结构，我们先看一下对它们的定义：

一种是 `eface`， 表示空接口，指未包含任何方法的接口。如定义一个变量 `var x interface{}`，还有我们经常使用的标准库中的 `func Println(a ...interface{}) (n int, err error) {}` 都必于 `eface` 类型，标准库中有许多这类的接口。

另一种是 `iface`，表示包含至少一个方法的接口。

根据两者的定义可知 `iface` 是 `eface` 的一个子集，那么为什么不直接使用`eface`呢? 主要原因还是为了做一些性能优化。

对数据结构的定义若不指定的话，默认位于 ` [src/runtime/runtime2.go](https://github.com/golang/go/blob/go1.15.6/src/runtime/runtime2.go#L208-L211) `。

这里需要提醒一下大家，对于 `doSomething(v interface{})` 这类的用法， **v 的类型是 interface{} 类型, 并非是任意类型**，这一点十分重要。当将一个值传递给函数的时候，go将会做类型转换操作，所有的值类型在运行时只有一种类型，`v` 的静态类型是 `interface`。

## eface 结构体 

我们看一下空接口的结构体定义

```
// src/runtime/runtime2.go
type eface struct {
	_type *_type
	data  unsafe.Pointer
}
```

`eface` 结构体主要包含两个字段，共16字节。
`_type`：这个是运行时 ` [runtime._type](https://github.com/golang/go/blob/go1.15.6/src/runtime/type.go#L27-L48) ` 指针类型，表示数据类型
`data`: 表示数据指针

runtime._type 结构定义如下

```
// src/runtime/type.go

// Needs to be in sync with ../cmd/link/internal/ld/decodesym.go:/^func.commonsize,
// ../cmd/compile/internal/gc/reflect.go:/^func.dcommontype and
// ../reflect/type.go:/^type.rtype.
// ../internal/reflectlite/type.go:/^type.rtype.
type _type struct {
	size       uintptr
	ptrdata    uintptr // size of memory prefix holding all pointers
	hash       uint32
	tflag      tflag
	align      uint8
	fieldAlign uint8
	kind       uint8
	// function for comparing objects of this type
	// (ptr to object A, ptr to object B) -> ==?
	equal func(unsafe.Pointer, unsafe.Pointer) bool
	// gcdata stores the GC type data for the garbage collector.
	// If the KindGCProg bit is set in kind, gcdata is a GC program.
	// Otherwise it is a ptrmask bitmap. See mbitmap.go for details.
	gcdata    *byte
	str       nameOff
	ptrToThis typeOff
}
```

字段说明
`size`: 存储的数据类型占用的空间大小，分配内存时需要这个信息
`ptrdata`:
`hash`: 快速判断确定类型是否相等
`equal`: 判断两个对象是否为相等
`gcdata`: 存储gc类型数据。

## iface 结构体 ![](https://blogstatic.haohtml.com/uploads/2021/02/7d44a96b1f356fcbd0d00c2597652db6-1.png)iface 结构体关系图

```
type iface struct {
	tab  *itab
	data unsafe.Pointer
}
```

同样包含两个字段，只有第一个字段不一样，这里是tab。iface 同 eface 一样，也是共占用16字节。

我们看下 ` [runtime.itab](https://github.com/golang/go/blob/go1.15.6/src/runtime/runtime2.go#L827-L837) ` 的数据结构

```
// layout of Itab known to compilers
// allocated in non-garbage-collected memory
// Needs to be in sync with
// ../cmd/compile/internal/gc/reflect.go:/^func.dumptabs.
type itab struct {
	inter *interfacetype
	_type *_type
	hash  uint32 // copy of _type.hash. Used for type switches.
	_     [4]byte
	fun   [1]uintptr // variable sized. fun[0]==0 means _type does not implement inter.
}

```

`runtime.itab` 结构体是接口类型中的核心组成部分，每个itab占用32字节。

`hash`: 字段拷贝自 _type.hash，用于类型切换。当我们想将 `interface` 类型转换成具体类型时，可以使用该字段快速判断目标类型和具体类型 [`runtime._type`][1] 是否一致
`fun`: 固定大小的数组。这里有个问题 fun 是长度为 1 的 uintptr 数组，那么怎么表示多个 method 呢？

你可能会觉得奇怪，为什么 `fun` 数组的大小为1，要是接口定义了多个方法可怎么办？实际上，这里存储的是第一个方法的函数指针。如果 `fun[0]` 为 `` 时，说明 `_type` 并没有实现该接口。否则表示已经实现，如果有更多的方法，在它之后的内存空间里继续存储。从汇编角度来看，通过增加地址就能获取到这些函数指针，没什么影响。顺便提一句，这些方法是按照函数名称的字典序进行排列的。

`inter` 是一个 [`interfacetype`](https://github.com/golang/go/blob/go1.15.6/src/runtime/type.go#L359-L363) 类型，类型定义如下：

```
type interfacetype struct {
	typ     _type
	pkgpath name
	mhdr    []imethod
}
```

`typ`: 接口类型
`pkgpath`: 包名
`mhdr`: 方法集列表

在go中除了`interfacetpe` 外，还有一些类似的类型，如 `arraytype`、`maptype`、`chartype`、`chantype` 和 `slicetype` 等,它们都可以在 ` [src/runtime/type.go](https://github.com/golang/go/blob/go1.15.6/src/runtime/type.go) ` 文件里找到。

Go语言各种数据类型都是在 `_type` 字段的基础上，增加一些额外的字段来进行管理的：

```
type arraytype struct {
	typ   _type
	elem  *_type
	slice *_type
	len   uintptr
}

type chantype struct {
	typ _type
	elem *_type
	dir uintptr
}

type slicetype struct {
	typ _type
	elem *_type
}

type functype struct {
	typ      _type
	inCount  uint16
	outCount uint16
}

type ptrtype struct {
	typ  _type
	elem *_type
}

type structtype struct {
	typ _type
	pkgPath name
	fields []structfield
}
type structfield struct {
	name       name
	typ        *_type
	offsetAnon uintptr
}
```

这些数据类型的结构体定义，是反射实现的基础。

`imethtod` 结构体定义

```
type imethod struct {
	name nameOff
	ityp typeOff
}
```

`nameOff` 和 `typeOff` 都是 int32 别名定义。

# 类型转换 

在实现一个接口时候，一般有有两种实现方式， 使用 `值接收者` 方法实现或者使用 `指针接收者` 实现，两者又有何区别呢？这里会涉及一些go汇编的语法，如果不太熟悉的话，建议先了解一下，阅读 [`这里`](https://chai2010.cn/advanced-go-programming-book/ch3-asm/ch3-01-basic.html)。

在类型转换时，将使用一系列 `convXXX` 函数。如空接口将调用 ` [convT2E](https://github.com/golang/go/blob/go1.15.6/src/runtime/iface.go#L311-L332) ` 系列函数，非空接口调用 ` [convT2I](https://github.com/golang/go/blob/go1.15.6/src/runtime/iface.go#L405-L418) ` 系列函数，定义这些函数正是Go内部为了性能考虑，避免对函数 `typedmemmove` 的调用（从函数名的最后一个字符也可以看出转换的是哪种接口）。

下面我们从一个示例中来确认这一点。

```
package main

import (
	"fmt"
)

// 定义有方法的接口，对应的是 iface 结构体
type MyInterface interface {
	Print()
}
type MyStruct struct{}

func (ms MyStruct) Print() {}

func main() {
	// 空接口，对应 eface
	a := 1
	var x interface{} = a

	// 非空方法的接口，对应 iface
	var b MyStruct
	var y MyInterface = b

	fmt.Println(x, y)
}

```

执行命令

```
go tool compile -S main.go
```

从输出结果中可以看到调用的 convXXX 函数。



# 参考资料 

 * [http://legendtkl.com/2017/07/01/golang-interface-implement/](http://legendtkl.com/2017/07/01/golang-interface-implement/)
 * [https://golang.org/doc/effective_go.html#interfaces_and_types](https://golang.org/doc/effective_go.html#interfaces_and_types)
 * [https://en.wikipedia.org/wiki/Duck_typing](https://en.wikipedia.org/wiki/Duck_typing)
 * [https://github.com/teh-cmc/go-internals/blob/master/chapter2_interfaces/README.md](https://github.com/teh-cmc/go-internals/blob/master/chapter2_interfaces/README.md)
 * [https://www.cnblogs.com/33debug/p/11867306.html](https://www.cnblogs.com/33debug/p/11867306.html)
 * [https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-interface/](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-interface/)
 * [https://mp.weixin.qq.com/s/vSgV_9bfoifnh2LEX0Y7cQ](https://mp.weixin.qq.com/s/vSgV_9bfoifnh2LEX0Y7cQ)
 * [https://www.zhihu.com/question/318138275/answer/699989214](https://www.zhihu.com/question/318138275/answer/699989214)
 * [https://www.jianshu.com/p/82436645927b](https://www.jianshu.com/p/82436645927b)

 [1]: https://github.com/golang/go/blob/go1.15.6/src/runtime/type.go#L27-L48