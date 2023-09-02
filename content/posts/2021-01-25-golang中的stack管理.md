---
title: Golang中Stack的管理
author: admin
type: post
date: 2021-01-25T04:07:10+00:00
url: /archives/21544
categories:
 - 程序开发
tags:
 - golang
 - stack

---
# 栈的演变 

在 Go1.13之前的版本，Golang 栈管理是使用的`分段栈（Segment Stacks）`机制来实现的，由于sgement stack 存在 `热分裂（hot split`）的问题，后面版本改为采用`连续栈（ [Contiguous stacks](https://docs.google.com/document/d/1wAaf1rYoM4S4gtnPh0zOlGzWtrZFQ5suE8qr2sD8uWQ/pub)）`机制( [说明](https://golang.org/doc/go1.3#stacks))。

## 分段栈（Segment Stack） 

分段栈是指开始时只有一个stack，当需要更多的 stack 时，就再去申请一个，然后将多个stack 之间用双向链接连接在一起。当使用完成后，再将无用的 `stack` 从链接中删除释放内存。![](https://blogstatic.haohtml.com/uploads/2021/01/d2b5ca33bd970f64a6301fa75ae2eb22-3.png)segment stack

可以看到这样确实实现了stack 按需增长和收缩，在增加新stack时不需要拷贝原来的数据，系统使用率挺高的。但在一定特别的情况下会存在 `热分裂（hot split)` 的问题。

当一个 stack 即将用完的时候，任意一个函数都会导致堆栈的扩容，当函数执行完返回后，又要触发堆栈的收缩。如果这个操作是在一个for语句里执行的话，则过多的 malloc 和 free 重复操作将导致系统资源开销非常的大。

如果你对 `Stack Frame` 不了解的话，可能先阅读一下 [Go 语言机制之栈和指针](https://studygolang.com/articles/12443)

## 连续栈（Contiguous stacks） ![](https://blogstatic.haohtml.com/uploads/2021/01/4e81521c30d4ccbdbac652573f27db33.png)Contiguous stacks 扩容与收缩

连续栈使用了另一种管理机制，每当一个stack 空间不够的时候，直接再申请一个2倍大小的空间，然后再将stack数据拷贝过去，同时修改指向原来stack 的指针到新stack，最后再将旧stack删除。

这种机制可以在当stack 空间快用尽的时候，避免在for语句里频繁触发扩容的问题。也正是官方采用这种机制的原因。

# 栈的初始化 

在上篇文章《 [Golang 的底层引导流程/启动顺序](https://blog.haohtml.com/archives/21411)》中介绍过，在应用启动时会有一系列的初始化工作，其中就包括对栈的初始化 ([源码][1]），调用函数 ` [stackinit()](https://github.com/golang/go/blob/go1.15.6/src/runtime/stack.go#L158-L170) `。

```
func stackinit() {
	if _StackCacheSize&_PageMask != 0 {
		throw("cache size must be a multiple of page size")
	}
	for i := range stackpool {
		stackpool[i].item.span.init()
		lockInit(&stackpool[i].item.mu, lockRankStackpool)
	}
	for i := range stackLarge.free {
		stackLarge.free[i].init()
		lockInit(&stackLarge.lock, lockRankStackLarge)
	}
}
```

这里有两个很重要的栈变量，一个是 `stackpool`，另一个是 `stackLarge`，两者均为 global pool 级别，实现为对stack的复用。前者是为每一种规格的大小以链接的形式存储可复用stack信息，共有 `_NumStackOrders` 类。后者为 `large stack` ，即大小>32K的全局栈。

在应用刚开始时，会分别对这两种全局stack变量进行初始化。

# 扩容 

函数 [`newstack()`](https://github.com/golang/go/blob/go1.15.6/src/runtime/stack.go#L928-L1082)。在编译时调用的是 `runtime.morestack()` 函数。

当G的堆栈空间不够用时，系统会再申请一块两倍大小( [源码](https://github.com/golang/go/blob/go1.15.6/src/runtime/stack.go#L1050-L1068))的空间，然后将原堆栈数据拷贝过去，再删除原来的堆栈。

步骤

 1. 申请两倍大小空间
 2. 修改`G`的状态，将 `_Grunning` 改为 `_Gcopystack`，函数 ` [casgstatus()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L804-L842) `
 3. 拷贝旧数据到新堆栈 ` [copystack()](https://github.com/golang/go/blob/go1.15.6/src/runtime/stack.go#L838-L917) `，并调整指针到新堆栈地址 ` [gentraceback()](https://github.com/golang/go/blob/go1.15.6/src/runtime/traceback.go?L=98) `，最后释放旧堆栈 ` [stackfree()](https://github.com/golang/go/blob/go1.15.6/src/runtime/stack.go#L421-L496) `
 4. 再恢复`G`的状态，将 `_Gcopystack` 为 `_Grunning`，函数 ` [casgstatus()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L804-L842) `

# 收缩 

函数 [`shrinkstack()`](https://github.com/golang/go/blob/go1.15.6/src/runtime/stack.go#L1120-L1179)

当一个 G 占用的stack非常大，后期却很少使用的stack，这时候则会有大量的stack处于空间状态，我们需要对空间进行收缩，以释放资源。

**收缩原则**

 * 在GC期间，如果一个goroutine未使用stack的大小占用超过X% ，则需要将其复制到一个small stack中。当需要的时候再进行扩容。
 * 在GC期间，如果一个goroutine未使用stack的大小占用超过X%，将其stack guard 降低到一个较小的值。如果它在下一次GC时没有受到保护，则将其复制到一个small stack 中。

目前的状态是在GC期间，如果一个goroutine使用大小小于其stack的`1/4`，则释放stack 底部的`1/2`。同样调用的是 ` [copystack()](https://github.com/golang/go/blob/go1.15.6/src/runtime/stack.go#L838-L917) ` 函数，此函数内会进行新栈的创建。

# 栈的释放 

对stack的申请与释放请参考 《 [goroutine栈的申请与释放](https://blog.haohtml.com/archives/30403)》

# 其它 

## [Stack frame layout](https://github.com/golang/go/blob/release-branch.go1.15/src/runtime/stack.go#L505) 

`Stack Frame` 是指函数运行时占用的内存空间，是栈上的数据集合，它包括：

 * Local variables
 * Saved copies of registers modified by subprograms that could need restoration
 * Argument parameters
 * Return address

**`FP`，`SP`，`PC` ，**`SB`

 * `FP`: Frame Pointer– Points to the bottom of the argument list
 * `SP`: Stack Pointer– Points to the top of the space allocated for local variables
 * `PC`: Program Counter –  jumps and branches
 * `SB`: Static base pointer – global symbols![ch3-3-arch-amd64-02.ditaa](https://blogstatic.haohtml.com/uploads/2021/01/7ff5eb752733f96a58687e8ab789f184-5.png)

所有用户空间的数据都可以通过FP/SP(局部数据、输入参数、返回值)和SB(全局数据)访问。 通常情况下，不会对SB/FP寄存器进行运算操作，通常情况以会以SB/FP/SP作为基准地址，进行偏移解引用 等操作。

伪SP是一个比较特殊的寄存器，因为还存在一个同名的SP真寄存器。**真SP寄存器对应的是栈的顶部**，一般用于定位调用其它函数的参数和返回值。

当需要区分伪寄存器和真寄存器的时候只需要记住一点：伪寄存器一般需要一个标识符和偏移量为前缀，如果没有标识符前缀则是真寄存器。比如`(SP)`、`+8(SP)`没有标识符前缀为真SP寄存器，而`a(SP)`、`b+8(SP)`有标识符为前缀表示伪寄存器。

```
// Stack frame layout
//
// (x86)
// +------------------+
// | args from caller |
// +------------------+ <- frame->argp
// |  return address  |
// +------------------+
// |  caller's BP (*) | (*) if framepointer_enabled && varp < sp
// +------------------+ <- frame->varp
// |     locals       |
// +------------------+
// |  args to callee  |
// +------------------+ <- frame->sp
//
// (arm)
// +------------------+
// | args from caller |
// +------------------+ <- frame->argp
// | caller's retaddr |
// +------------------+ <- frame->varp
// |     locals       |
// +------------------+
// |  args to callee  |
// +------------------+
// |  return address  |
// +------------------+ <- frame->sp
```

可以看到 `x86` 和 `arm` 架构布局是不一样的。目前来说，我们一般只关注 `x86` 架构布局就可以了，必须arm还不是主流。

## 调用栈 

这里以一个函数调用过程A->B->C为例了来解释调用栈过程![v2-03c1e190354612a14c039764ca9f7c4f_720w](https://blogstatic.haohtml.com/uploads/2021/01/743822e66bac81e5fc012e35000a8467-1.jpg)

分配从高到低顺序进行。

推荐阅读针对 heap 的GC： [Go：内存管理与内存清理](https://studygolang.com/articles/27144),

# 参考 

 * [https://draveness.me/system-design-memory-management/](https://draveness.me/system-design-memory-management/)
 * [https://golang.design/under-the-hood//zh-cn/part2runtime/ch06sched/stack/](https://golang.design/under-the-hood//zh-cn/part2runtime/ch06sched/stack/)
 * [https://studygolang.com/articles/2917](https://studygolang.com/articles/2917)
 * [https://guidao.github.io/asm.html](https://guidao.github.io/asm.html)
 * [https://segmentfault.com/a/1190000019753885](https://segmentfault.com/a/1190000019753885)
 * [https://zhuanlan.zhihu.com/p/56750445](https://zhuanlan.zhihu.com/p/56750445)
 * [Go 语言机制之栈和指针][2]
 * [Contiguous stacks][3]
 * [聊一聊goroutine stack][4]
 * [Go: Goroutine 的堆栈大小是如何演进的][5]
 * [Go：内存管理与内存清理](https://studygolang.com/articles/27144)
 * [https://www.jianshu.com/p/20c1855d1b9c](https://www.jianshu.com/p/20c1855d1b9c)
 * [https://segmentfault.com/a/1190000019570427](https://segmentfault.com/a/1190000019570427)
 * [https://github.com/jukylin/blog/blob/master/Go%E6%BA%90%E7%A0%81%EF%BC%9A%E5%8D%8F%E7%A8%8B%E6%A0%88.md](https://github.com/jukylin/blog/blob/master/Go%E6%BA%90%E7%A0%81%EF%BC%9A%E5%8D%8F%E7%A8%8B%E6%A0%88.md)
 * [https://medium.com/a-journey-with-go/go-how-does-the-goroutine-stack-size-evolve-447fc02085e5](https://medium.com/a-journey-with-go/go-how-does-the-goroutine-stack-size-evolve-447fc02085e5) ( [中文](https://www.jianshu.com/p/51834ff8b58b))
 * [https://docs.google.com/document/d/13v_u3UrN2pgUtPnH4y-qfmlXwEEryikFu0SQiwk35SA/pub](https://docs.google.com/document/d/13v_u3UrN2pgUtPnH4y-qfmlXwEEryikFu0SQiwk35SA/pub)
 * [https://docs.google.com/document/d/1un-Jn47yByHL7I0aVIP_uVCMxjdM5mpelJhiKlIqxkE/edit#heading=h.bvezjdnoi4no](https://docs.google.com/document/d/1un-Jn47yByHL7I0aVIP_uVCMxjdM5mpelJhiKlIqxkE/edit#heading=h.bvezjdnoi4no)

 [1]: https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L562
 [2]: https://studygolang.com/articles/12443
 [3]: https://docs.google.com/document/d/1wAaf1rYoM4S4gtnPh0zOlGzWtrZFQ5suE8qr2sD8uWQ/pub
 [4]: https://zhuanlan.zhihu.com/p/28409657
 [5]: https://studygolang.com/articles/28971