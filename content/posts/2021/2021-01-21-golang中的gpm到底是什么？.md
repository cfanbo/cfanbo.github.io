---
title: golang中G、P、M 和 sched 三者的数据结构
author: admin
type: post
date: 2021-01-21T02:54:12+00:00
url: /archives/21010
categories:
 - 程序开发
tags:
 - golang
 - gpm

---
G、P、M 三者是golang实现高并发能的最为重要的概念，`runtime` 通过 `调度器` 来实现三者的相互调度执行，通过 `p` 将用户态的 `g` 与内核态资源 `m` 的动态绑定来执行，以减少以前通过频繁创建内核态线程而产生的一系列的性能问题，充分发挥服务器最大有限资源。![](https://blogstatic.haohtml.com/uploads/2021/01/7c68d000148bf601267b43631c795bfd.png)GPM 协作

调度器的工作是将一个 G（需要执行的代码）、一个 M（代码执行的地方）和一个 P（代码执行所需要的权限和资源）结合起来。

所有的 g、m 和 p 对象都是分配在`堆`上且永不释放的，所以它们的内存使用是很稳定的。得益于此，runtime 可以在调度器实现中避免写屏障。当一个G执行完成后，可以放入pool中被再次使用，避免重复申请资源。

本节主要通过阅读runtime源码来认识这三个组件到底长的是什么样子，以此加深对 GPM 的理解。go version go1.15.6

理解下文前建议先阅读一下 `src/runtime/HACKING.md` 文件，中文可阅读 [这里](https://www.purewhite.io/2019/11/28/runtime-hacking-translate/)，这个文件内容是面向开发者理解`runtime`的很值得看一看。

本文若没有指定源码文件路径，则默认为 `src/runtime/runtime2.go`。

# G 

G是英文字母`goroutine`的缩写，一般称为“`协程`”，其实这个词还是无法完整表达它的意思的，但这用的人的多了就成了统称。注意它与线程和进程的区别，这个应该很容易理解，每个gopher应该都知道。

每个 Goroutine 对应一个 `g` 结构体，它有自己的栈内存, G 存储 Goroutine 的运行堆栈、状态以及任务函数，可重复用，

Goroutine数据结构位于 `src/runtime/runtime2.go` 文件，注意此文件里有太多重要的底层数据结构，对于我们理解底层runtime非常的重要，建议大量多看看。不需要记住每一个数据结构，但需要的时候要能第一时间想到在哪里查找。

当一个 goroutine 退出时，`g` 对象会被放到一个空闲的 `g` 对象池中以用于后续的 goroutine 的使用， 以减少内存分配开销。

Goroutine 字段非常的多，我们这里分段来理解

```
type g struct {
	// Stack parameters.
	// stack describes the actual stack memory: [stack.lo, stack.hi).
	// stackguard0 is the stack pointer compared in the Go stack growth prologue.
	// It is stack.lo+StackGuard normally, but can be StackPreempt to trigger a preemption.
	// stackguard1 is the stack pointer compared in the C stack growth prologue.
	// It is stack.lo+StackGuard on g0 and gsignal stacks.
	// It is ~0 on other goroutine stacks, to trigger a call to morestackc (and crash).
	stack       stack   // offset known to runtime/cgo

	// 检查栈空间是否足够的值, 低于这个值会扩张栈, 0是go代码使用的
	stackguard0 uintptr // offset known to liblink

	// 检查栈空间是否足够的值, 低于这个值会扩张栈, 1是原生代码使用的
	stackguard1 uintptr // offset known to liblink
}
```

`stack` 描述了当前 `Goroutine` 的栈内存范围`[stack.lo, stack.hi)`，其中stack 的数据结构为

```
// Stack describes a Go execution stack.
// The bounds of the stack are exactly [lo, hi),
// with no implicit data structures on either side.
// 描述go执行栈
// 栈边界为[lo, hi)，左包含可不包含，即 lo≤stack<hi
// 两边都没有隐含的数据结构。
type stack struct {
	lo uintptr // 该协程拥有的栈低位
	hi uintptr // 该协程拥有的栈高位
}
```

`stackguard0` 和 `stackguard1` 均是一个栈指针，用于扩容场景，前者用于 Go stack ，后者用于C stack。

如果 `stackguard0` 字段被设置成 `StackPreempt` 意味着当前 Goroutine 发出了抢占请求。

在`g`结构体中的`stackguard0` 字段是出现爆栈前的警戒线。`stackguard0`的偏移量是`16`个字节，与当前的真实`SP(stack pointer)`和爆栈警戒线（`stack.lo+StackGuard`）比较，如果超出警戒线则表示需要进行栈扩容.先调用`runtime·morestack_noctxt()`进行栈扩容，然后又跳回到函数的开始位置，此时此刻函数的栈已经调整了。然后再进行一次栈大小的检测，如果依然不足则继续扩容，直到栈足够大为止。对于StackGuard 的介绍可以参考 [这里](https://jiajunhuang.com/articles/2020_08_26-stackguard.md.html)。

关于对 Stack 的理解可参考这篇 [文章](https://studygolang.com/articles/12443)

```
type g struct {
	preempt       bool // preemption signal, duplicates stackguard0 = stackpreempt
	preemptStop   bool // transition to _Gpreempted on preemption; otherwise, just deschedule
	preemptShrink bool // shrink stack at synchronous safe point
}
```

`preempt` 抢占标记，其值为true 执行 stackguard0 = stackpreempt
`preemptStop` 将抢占标记修改为 _Gpreedmpted，如果修改失败则取消
`preemptShrink` 在同步安全点收缩栈

```
type g struct {
	_panic       *_panic // innermost panic - offset known to liblink
	_defer       *_defer // innermost defer
}
```

`_panic` 当前Goroutine 中的panic
`_defer` 当前Goroutine 中的defer

```
type g struct {
	m            *m      // current m; offset known to arm liblink
	sched        gobuf
	goid         int64
}
```

`m` 当前 Goroutine 绑定的M
`sched` 存储当前 Goroutine 调度相关的数据，上下方切换时会把当前信息保存到这里，用的时候再取出来，它的用途可参考函数 ` [newproc1()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L3566-L3674) `。
`goid` 当前 Goroutine 的唯一标识，对开发者不可见，一般不使用此字段。可参考相关文章了解为什么Go开发团队为什么不向外开放访问此字段。

gobuf 结构体

```
type gobuf struct {
	// The offsets of sp, pc, and g are known to (hard-coded in) libmach.
	// 寄存器 sp,pc和g的偏移量，硬编码在libmach
	//
	// ctxt is unusual with respect to GC: it may be a
	// heap-allocated funcval, so GC needs to track it, but it
	// needs to be set and cleared from assembly, where it's
	// difficult to have write barriers. However, ctxt is really a
	// saved, live register, and we only ever exchange it between
	// the real register and the gobuf. Hence, we treat it as a
	// root during stack scanning, which means assembly that saves
	// and restores it doesn't need write barriers. It's still
	// typed as a pointer so that any other writes from Go get
	// write barriers.
	sp   uintptr
	pc   uintptr
	g    guintptr
	ctxt unsafe.Pointer
	ret  sys.Uintreg
	lr   uintptr
	bp   uintptr // for GOEXPERIMENT=framepointer
}
```

`sp` 栈指针位置
`pc` 程序计数器，运行到的程序位置
`gobuf` 主要存储一些寄存器信息，如`sp`、`pc` 和 `g` 的偏移量，硬编码在libmach
`ctxt` 不常见，可能是一个分配在heap的函数变量，因此GC 需要追踪它，不过它有可能需要设置并进行清除，在有`写屏障` 的时候有些困难。重点了解一下 `write barriers`
`g` 技能当前 `gobuf` 的 Goroutine
`ret` 系统调用的结果
`bp` 未知

调度器在将 G 由一种状态变更为另一种状态时，需要将上下文信息保存到这个`gobuf`结构体，当再次运行 G 的时候，再从这个结构体中读取出来，主要用来暂时上下文信息。其中的栈指针和程序计数器会用来存储或者恢复寄存器中的值，改变程序即将执行的代码。

Goroutine 的状态有以下几种（ [源码](https://github.com/golang/go/blob/go1.15.6/src/runtime/runtime2.go#L16-L86)）

| 状态 | 描述 |
| ------------------- | -------------------------------------------------------------------------------------------------- |
| `_Gidle` | 0 刚刚被分配并且还没有被初始化 |
| `_Grunnable` | 1 没有执行代码，没有栈的所有权，存储在运行队列中 |
| `_Grunning` | 2 可以执行代码，拥有栈的所有权，被赋予了内核线程 M 和处理器 P |
| `_Gsyscall` | 3 正在执行系统调用，没有执行用户代码，拥有栈的所有权，被赋予了内核线程 M 但是不在运行队列上 |
| `_Gwaiting` | 4 由于运行时而被阻塞，没有执行用户代码并且不在运行队列上，但是可能存在于 Channel 的等待队列上。若需要时执行ready()唤醒。 |
| `_Gmoribund_unused` | 5 当前此状态未使用，但硬编码在了gdb 脚本里，可以不用关注 |
| `_Gdead` | 6 没有被使用，可能刚刚退出，或在一个freelist；也或者刚刚被初始化；没有执行代码，可能有分配的栈也可能没有；G和分配的栈（如果已分配过栈）归刚刚退出G的M所有或从free list 中获取 |
| `_Genqueue_unused` | 7 目前未使用，不用理会 |
| `_Gcopystack` | 8 栈正在被拷贝，没有执行代码，不在运行队列上 |
| `_Gpreempted` | 9 由于抢占而被阻塞，没有执行用户代码并且不在运行队列上，等待唤醒 |
| `_Gscan` | 10 GC 正在扫描栈空间，没有执行代码，可以与其他状态同时存在 |

**Goroutine 的状态**

需要注意的是对于 `_Gmoribund_unused` 状态并未使用，但在 `gdb` 脚本中存在；而对于 \_Genqueue\_unused 状态目前也未使用，不需要关心。

`_Gscan` 与上面除了`_Grunning` 状态以外的其它状态相组合，表示 `GC` 正在扫描栈。Goroutine 不会执行用户代码，且栈由设置了 `_Gscan` 位的 Goroutine 所有。

| 状态 | 描述 |
| ----------------- | ---------------------------------- |
| `_Gscanrunnable` | = \_Gscan + \_Grunnable // 0x1001 |
| `_Gscanrunning` | = \_Gscan + \_Grunning // 0x1002 |
| `_Gscansyscall` | = \_Gscan + \_Gsyscall // 0x1003 |
| `_Gscanwaiting` | = \_Gscan + \_Gwaiting // 0x1004 |
| `_Gscanpreempted` | = \_Gscan + \_Gpreempted // 0x1009 |

**Goroutine 的状态**

可以看到除了上面提到的两个未使用的状态外一共有14种状态值。许多状态之间是可以进行改变的。如下图所示![](https://blogstatic.haohtml.com/uploads/2021/01/15b7d06510d9b3b1ec31d7a5cbfec9c5.png)goroutine status ( [https://github.com/golang-design/Go-Questions](https://github.com/golang-design/Go-Questions))![](https://blogstatic.haohtml.com/uploads/2021/01/3158987106a957636da1fd9d0f8f9509.png)G的状态机流转图

```
type g strcut {
	syscallsp    uintptr        // if status==Gsyscall, syscallsp = sched.sp to use during gc
	syscallpc    uintptr        // if status==Gsyscall, syscallpc = sched.pc to use during gc
	stktopsp     uintptr        // expected sp at top of stack, to check in traceback
	param        unsafe.Pointer // passed parameter on wakeup
	atomicstatus uint32
	stackLock    uint32 // sigprof/scang lock; TODO: fold in to atomicstatus
}
```

`atomicstatus` 当前G的状态，上面介绍过G的几种状态值
`syscallsp` 如果G 的状态为 `Gsyscall` ,那么值为 `sched.sp` 主要用于GC 期间
`syscallpc` 如果G的状态为 `GSyscall` ，那么值为 `sched.pc` 同上也是用于GC 期间，由此可见这两个字段是一起使用的
`stktopsp` 用于回源跟踪，如何理解？
`param` 唤醒G时传入的参数，如调用 `ready()`
`stackLock` 栈锁，什么场景下会使用？

```
type g struct {
	waitsince    int64      // approx time when the g become blocked
	waitreason   waitReason // if status==Gwaiting
}
```

`waitsince` G 阻塞时长
`waitreason` 阻塞原因

```
type g struct {
	// asyncSafePoint is set if g is stopped at an asynchronous
	// safe point. This means there are frames on the stack
	// without precise pointer information.
	asyncSafePoint bool

	paniconfault bool // panic (instead of crash) on unexpected fault address
	gcscandone   bool // g has scanned stack; protected by _Gscan bit in status
	throwsplit   bool // must not split stack
}
```

`asyncSafePoint` 异步安全点；如果 g 在`异步安全点`停止则设置为`true`，表示在栈上没有精确的指针信息
`paniconfault` 地址异常引起的panic（代替了崩溃）
`gcscandone` g 扫描完了栈，受状态 `_Gscan` 位保护
`throwsplit` 不允许拆分stack 什么意思？

```
type g struct {
	// activeStackChans indicates that there are unlocked channels
	// pointing into this goroutine's stack. If true, stack
	// copying needs to acquire channel locks to protect these
	// areas of the stack.
	activeStackChans bool
	// parkingOnChan indicates that the goroutine is about to
	// park on a chansend or chanrecv. Used to signal an unsafe point
	// for stack shrinking. It's a boolean value, but is updated atomically.
	parkingOnChan uint8
}
```

`activeStackChans` 表示是否有未加锁定的channel指向到了g 栈，如果为true,那么对栈的复制需要channal锁来保护这些区域
`parkingOnChan` 表示g 是放在chansend 还是 chanrecv。用于栈的收缩，是一个布尔值，原子更新

```
type g struct {
	raceignore     int8     // ignore race detection events
	sysblocktraced bool     // StartTrace has emitted EvGoInSyscall about this goroutine
	sysexitticks   int64    // cputicks when syscall has returned (for tracing)
	traceseq       uint64   // trace event sequencer
	tracelastp     puintptr // last P emitted an event for this goroutine
	lockedm        muintptr
	sig            uint32
	writebuf       []byte
	sigcode0       uintptr
	sigcode1       uintptr
	sigpc          uintptr
	gopc           uintptr         // pc of go statement that created this goroutine
	ancestors      *[]ancestorInfo // ancestor information goroutine(s) that created this goroutine (only used if debug.tracebackancestors)
	startpc        uintptr         // pc of goroutine function
	racectx        uintptr
	waiting        *sudog         // sudog structures this g is waiting on (that have a valid elem ptr); in lock order
	cgoCtxt        []uintptr      // cgo traceback context
	labels         unsafe.Pointer // profiler labels
	timer          *timer         // cached timer for time.Sleep
	selectDone     uint32         // are we participating in a select and did someone win the race?
}
```

`gopc` 创建当前G的pc
`startpc` go func 的pc
`waiting` 如何理解？
`timer` 通过time.Sleep 缓存 timer

从字段命名来看，许多字段都与trace 有关，不清楚什么意思

```
type g struct {
	// Per-G GC state

	// gcAssistBytes is this G's GC assist credit in terms of
	// bytes allocated. If this is positive, then the G has credit
	// to allocate gcAssistBytes bytes without assisting. If this
	// is negative, then the G must correct this by performing
	// scan work. We track this in bytes to make it fast to update
	// and check for debt in the malloc hot path. The assist ratio
	// determines how this corresponds to scan work debt.
	gcAssistBytes int64
}
```

`gcAssistBytes` 与GC相关。

为了保证用户程序分配内存的速度不会超出后台任务的标记速度，运行时还引入了标记辅助技术，它遵循一条非常简单并且朴实的原则，**分配多少内存就需要完成多少标记任务**。每一个 Goroutine 都持有 `gcAssistBytes` 字段，这个字段存储了当前 Goroutine 辅助标记的对象字节数。在并发标记阶段期间，当 Goroutine 调用 [`runtime.mallocgc`][1] 分配新对象时，该函数会检查申请内存的 Goroutine 是否处于入不敷出的状态。

**总结**

 * 每个 G 都有自己的状态，状态保存在 `atomicstatus` 字段，共有十几种状态值。
 * 每个 G 在状态发生变化时，即 `atomicstatus` 字段值被改变时，都需要保存当前G的上下文的信息，这个信息存储在 `sched` 字段，其数据类型为`gobuf`，想理解存储的信息可以看一下这个结构体的各个字段
 * 每个 G 都有三个与抢占有关的字段，分别为 `preempt`、`preemptStop` 和 `premptShrink`
 * 每个 G 都有自己的唯一id, 字段为`goid`，但此字段官方不推荐开发使用
 * 每个 G 最多可以绑定一个`m`，如果未绑定，则值为 `nil`
 * 每个 G 都有自己内部的 `defer` 和 `panic`。
 * G 可以被阻塞，并存储有阻塞原因，字段 `waitsince` 和 `waitreason`
 * G 可以被进行 GC 扫描，相关字段为 `gcscandone`、`atomicstatus` （ `_Gscan` 与上面除了`_Grunning` 状态以外的其它状态组合）

# P 

P 表示逻辑处理器，对 G 来说，P 相当于 CPU 核，G 只有绑定到 P 才能被调度。对 M 来说，P 提供了相关的执行环境(Context)，如内存分配状态(mcache)，任务队列(G)等。

P 的数量决定了系统内最大可并行的 G 的数量（前提：逻辑 CPU 核数  >= P 的数量）。推荐使用默认值，过多的设置P的个数可能会导致上下文的切换成本。

另外在容器应用里，P 的默认值是读取的 `/proc/cpuinfo` 文件，因此如果在启动容器时指定了cpu个数，则会导致实现P的个数>逻辑cpu个数，这样容易产生上下方切换成本过高问题，此时建议在程序中使用三方库，如 [https://github.com/uber-go/automaxprocs](https://github.com/uber-go/automaxprocs) 来解决此类问题。

P的数据结构也有几十个字段，我们将其分开来理解

```
type p struct {
	id          int32
	status      uint32 // one of pidle/prunning/...
	link        puintptr
	schedtick   uint32     // incremented on every scheduler call
	syscalltick uint32     // incremented on every system call
	sysmontick  sysmontick // last tick observed by sysmon
}
```

`id`: P的唯一标识
`status` P当前状态，状态值有`_Pidle`、`_Prunning`、`_Psyscall`、`_Pgcstop` 和 `_Pdead`
`link` 未知
`schedtick` 每当被调度时递增
`syscalltick` 每当系统调用时递增
`sysmontick` sysmon 最后tick的时间，是一个 ` [sysmontick](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L4762) ` 数据类型。sysmon介绍： [https://www.jianshu.com/p/469d0c7a7936](https://www.jianshu.com/p/469d0c7a7936)

对于P的状态有五种：

| 状态 | 描述 |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `_Pidle` | 处理器没有运行用户代码或者调度器，被空闲队列或者改变其状态的结构持有，运行队列为空；也有可能是几种状态正在过度中状态 |
| `_Prunning` | 被线程 M 持有，并且正在执行用户代码或者调度器。只能由拥有当前P的M才可能修改此状态。M可以将P的状态修改为`_Pidle`（无工作可做）、`_Psyscall`(系统调用) 或 `_Pgstop`(GC); 另外M也可以P的使用权交给另一个M（调度一个锁定状态的G） |
| `_Psyscall` | 当前P没有执行用户代码，当前线程陷入系统调用 |
| `_Pgcstop` | 被线程 M 持有，当前处理器由于垃圾回收被停止，由 `_Prunning` 变为 `_Pgcstop` |
| `_Pdead` | 当前处理器已经不被使用，如通过动态调小 `GOMAXPROCS` 进行 `P` 收缩 |

**P 的状态**

```
type p struct {
	m           muintptr   // back-link to associated m (nil if idle)
	mcache      *mcache
	pcache      pageCache
	raceprocctx uintptr
}
```

`m` 当前正在绑定的m, 有可能为空，如`_Pidle`
`mcache` 每个p的小对象缓存，无锁，对应 ` [mcache](https://github.com/golang/go/blob/go1.15.6/src/runtime/mcache.go#L19-L54) ` 结构体
`pcache` 页面缓存，对应 ` [pageCache](https://github.com/golang/go/blob/go1.15.6/src/runtime/mpagecache.go#L14-L99) ` 结构体，不需要锁
`raceprocctx` race相关

其中 `mcache` 和 `pcache` 全是缓存相关字段，两个都是无锁结构体。

mcache是为了当G与P关联后，执行go code时，会为一些小对象（<32K）分配内存，这时直接从P.mcache 申请，避免直接从os申请，这样就允许多个P并发执行，减少申请内存的锁粒度，参考 [这里](https://medium.com/a-journey-with-go/go-memory-management-and-allocation-a7396d430f44)。

```
type p struct {
	deferpool    [5][]*_defer // pool of available defer structs of different sizes (see panic.go)
	deferpoolbuf [5][32]*_defer
}
```

`deferpool` 不同大小的defer，二维数组。具体见 panic.go 文件,有对此字段的一些处理逻辑
`deferpoolbuf` 同上

这两个字段是与 defer 相关

```
type p struct {
	// Cache of goroutine ids, amortizes accesses to runtime·sched.goidgen.
	goidcache    uint64
	goidcacheend uint64
}
```

`goidcache` goid 缓存
`goidcacheend` goid 缓存

两个都是 goroutine ids 的缓存。

```
type p struct {
	// Queue of runnable goroutines. Accessed without lock.
	runqhead uint32
	runqtail uint32
	runq     [256]guintptr

	// runnext, if non-nil, is a runnable G that was ready'd by
	// the current G and should be run next instead of what's in
	// runq if there's time remaining in the running G's time
	// slice. It will inherit the time left in the current time
	// slice. If a set of goroutines is locked in a
	// communicate-and-wait pattern, this schedules that set as a
	// unit and eliminates the (potentially large) scheduling
	// latency that otherwise arises from adding the ready'd
	// goroutines to the end of the run queue.
	runnext guintptr
}
```

`runqhead` 运行队列头
`runqtail` 运行队列尾
`runq` 运行队列, 数组类型，最大值为**256**

每个P都有一个自己的`runq`，除了自身有的runq 还有一个全局的runq, 对于每个了解过GPM的gohper应该都知道这一点。每个P下面runq的允许的最大goroutine 数量为256。

`runnext` 当前P（进入运行状态时）立即要运行的goroutine，可能为`nil`。如果此字段不为`nil` 的话，则表示下次即将运行的 goroutine，将直接从 `runnext` 字段取，而不必从 `runq` 中获取，因此其优先级高于`runq`。如果当前G 还有剩余的可用时间，那么就运行这个 runnext 的G 继承剩下的时间。此字段的调用请参考函数 [runtime.runqget()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L5261-L5288)。

这个字段是用来实现调度器亲和性的，我们知道原来一个G阻塞时，这时P会再获取一个G进行绑定执行，如果这时原来的G执行阻塞结束后，如果想再次被接着继续执行，就需要重新在P的 `runq` 进行排队，当 `runq` 里有太多的goroutine 时将会导致这个刚刚被解除阻塞的G迟迟无法得到执行，同时还有可能被其他处理器所窃取。从 Go 1.5 开始得益于 `P` 的特殊属性，从阻塞 channel 返回的 Goroutine 会优先运行，这里只需要将这个G放在 `runnext` 这个字段即可。参考文章 [理解 Go 并发以及调度器亲和性](https://mp.weixin.qq.com/s/XnqF5aZ_0F3-cZPUm9pM7w)

上面介绍了每个 P 的runq最大可以有`256`个goroutine, 再加上这个 `runnext` 字段的话，一个P最大的情况下可以有257个goroutine了。

```
type p struct {
	// Available G's (status == Gdead)
	gFree struct {
		gList
		n int32
	}
}
```

gFree 结构体表示空闲G的信息, 而其中 `gFree.n` 表示空闲G的个数。此结构体主要是为了方便实现对G的复用（只有当状态为 Gdead 时才有效）。关注一下 匿名结构体内的 ` [gList](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L5435-L5467) ` 字段, 这个字段在上篇 [文章《Golang中channel实现原理源码分析》](https://blog.haohtml.com/archives/20760) 里也介绍过它的使用场景。

```
type p struct {
	sudogcache []*sudog
	sudogbuf   [128]*sudog
}
```

`sudogcache` 缓存相关，slice 类型，*sudog 数据结构
`sudogbuf` 数组类型，*sudog 缓冲区

与*sudog 相关，可以看出与goroutine相关。不清楚这两个字段与上面的 runq 作用是什么。

```
type p strcut {
	// Cache of mspan objects from the heap.
	mspancache struct {
		// We need an explicit length here because this field is used
		// in allocation codepaths where write barriers are not allowed,
		// and eliminating the write barrier/keeping it eliminated from
		// slice updates is tricky, moreso than just managing the length
		// ourselves.
		len int
		buf [128]*mspan
	}
}
```

`mspancache` 从堆中缓存mspan对象。mspan是什么？

```
type p struct {
	tracebuf traceBufPtr

	// traceSweep indicates the sweep events should be traced.
	// This is used to defer the sweep start event until a span
	// has actually been swept.
	traceSweep bool
	// traceSwept and traceReclaimed track the number of bytes
	// swept and reclaimed by sweeping in the current sweep loop.
	traceSwept, traceReclaimed uintptr
}
```

与trace相关

```
type p struct {
	palloc persistentAlloc // per-P to avoid mutex

	_ uint32 // Alignment for atomic fields below

	// The when field of the first entry on the timer heap.
	// This is updated using atomic functions.
	// This is 0 if the timer heap is empty.
	timer0When uint64
}
```

`palloc` ？？？
`_` 为了下面字段原子操作而进行的内存对齐填充
`timer0When` 此字段由原子操作函数执行，如果 timer heap 为空，则值为 0。它表示P的 timer 定时器小堆第一个对象的when字段值，即最先执行的时间值

```
type p struct {
	// Per-P GC state
	gcAssistTime         int64    // Nanoseconds in assistAlloc
	gcFractionalMarkTime int64    // Nanoseconds in fractional mark worker (atomic)
	gcBgMarkWorker       guintptr // (atomic)
	gcMarkWorkerMode     gcMarkWorkerMode

	// gcMarkWorkerStartTime is the nanotime() at which this mark
	// worker started.
	gcMarkWorkerStartTime int64

	// gcw is this P's GC work buffer cache. The work buffer is
	// filled by write barriers, drained by mutator assists, and
	// disposed on certain GC state transitions.
	gcw gcWork

	// wbBuf is this P's GC write barrier buffer.
	//
	// TODO: Consider caching this in the running G.
	wbBuf wbBuf

	runSafePointFn uint32 // if 1, run sched.safePointFn at next safe point
}
```

P 的 GC 状态

`gcMarkWrokderStartTime` gc的开始时间
`gcw` GC work的 buffer, 写屏障填充，结构体 gcWork
`wbBuf` P的GC写屏障buffer
`runSafePointFn` 如果为1，则在下一个安全点运行 `sched.safePointFn`

各种状态的切换图![](https://blogstatic.haohtml.com/uploads/2021/01/0d20dfce0e3dd6968aebe84535b853c6.png)p status ( [https://github.com/golang-design/Go-Questions](https://github.com/golang-design/Go-Questions))

```
type p strcut {
	// Lock for timers. We normally access the timers while running
	// on this P, but the scheduler can also do it from a different P.
	timersLock mutex

	// Actions to take at some time. This is used to implement the
	// standard library's time package.
	// Must hold timersLock to access.
	timers []*timer

	// Number of timers in P's heap.
	// Modified using atomic instructions.
	numTimers uint32

	// Number of timerModifiedEarlier timers on P's heap.
	// This should only be modified while holding timersLock,
	// or while the timer status is in a transient state
	// such as timerModifying.
	adjustTimers uint32

	// Number of timerDeleted timers in P's heap.
	// Modified using atomic instructions.
	deletedTimers uint32

	// Race context used while executing timer functions.
	timerRaceCtx uintptr
}
```

`timerLock` timer锁
`timers` timer指针切片类型，timer 是标准库结构
`numTimers` P堆中的timer数，原子指令修改
`adjustTimers` p堆中 `timerModifiedEarlier` timer 的数量,修改时必须持有timerLock锁， 或者当定时器状态处于瞬态时，例如定时器调整。
`deletedTimers` 当前中p 堆中的 `deletedTimers` 计数器数量，原子指令修改
`timerRaceCtx` 执行计时器函数时使用的竞争上下文

上方 `timer0When`、`timerModifiedEarliest`、`timersLock`、`timers`、`numTimers`、`adjustTimers`、`deletedTimers` 、`timerDeleted` 和 `timerModifiedEarlier`，与其它几个相关的 timer 字段均与定时器有关。

```
type p struct {
	// preempt is set to indicate that this P should be enter the
	// scheduler ASAP (regardless of what G is running on it).
	preempt bool

	pad cpu.CacheLinePad
}
```

`preempt` 抢占标记，如果值为true，表示 p 进入调度（不管G在运行什么），关于抢占可以参考 [《Golang 基于信号的异步抢占与处理》](https://blog.haohtml.com/archives/23854)
`pad` cache line 对齐优化。不了解可参考文章 [CPU缓存体系对Go程序的影响](https://mp.weixin.qq.com/s/vnm9yztpfYA4w-IM6XqyIA)

**总结**

 * 每个P都有自己的状态，分别为 `_Pidle`、 `_Prunning` 、`_Psyscall` 、`_Pgcstop` 、`_Pdead`
 * 每个P都存储有自己被`调度次数`和`系统调用`的次数，字段 `schedtick` 和 `syscalltick`
 * P 可以绑定一个M。但也可以不绑定，这时m值为`nil`，字段`m`
 * 每个P 都有一个自己的`runq`,用来存放可以 `runnable` 状态的 goroutines， 最多可以存放256个 goroutine。一般在介绍GMP关系时，我们称之为 local queue 或 LRQ，当然还有一个global queue, 有时候也简写成 GRQ
 * 每个P都可能有一个 `runnext` 的 goroutine。如果此字段不为nil,则P下次执行G的时候，优先执行此字段的goroutine
 * P可以缓存goroutine，字段 `goidcache`
 * P的GC状态
 * P可以有多个timer, 以slice 形式存储， 字段 `timers`
 * P可以缓存堆上面的`mspan`对象，mspan对象是什么？
 * P 有一个抢占标记，字段为 `preempt`。如果为ture ，则表示P立即进入调度
 * P结构体使用了pad, 以优化cpu, 解决 [cpu伪共享](https://www.xwxwgo.com/post/2019/07/09/golang%E5%92%8Cfalse-sharing/) 的问题

# M 

M 是指OS 内核线程，代表着真正执行计算的资源，在绑定有效的 P 后，进入 schedule 循环；而 schedule 循环的机制大致是从 Global 队列、P 的 Local 队列以及 wait 队列中获取。

M 在runtime中对应的是 `m` 结构体。

**M 的数量是不定的，由 Go Runtime 调整，**为了防止创建过多 OS 线程导致系统调度不过来，目前默认最大限制为 `10000` 个。如果一个M工作完成后，找不到可用的P，则需要将自己休眠，并放在空闲线程中，等待下次使用。

切记：M 并不保留 G 状态，这是 G 可以跨 M 调度的基础。

下面对 m 的结构体做下介绍

```
type m struct {
	g0      *g     // goroutine with scheduling stack
	morebuf gobuf  // gobuf arg to morestack
	divmod  uint32 // div/mod denominator for arm - known to liblink
}
```

`g0` 这是一个很特殊的goroutine, 它是一个具有调度堆栈的能力, 参考 [这里](https://studygolang.com/articles/28443) 、 [这里](https://studygolang.com/articles/30251) 或 [这里](https://blog.haohtml.com/archives/22353)
`morebuf` 堆栈扩容使用([见这里][2])，gobuf 数据类型，gobuf这个数据结构在g结构体中已出现过，它的作用就是保存一个g 的上下文数据。这里作为传递给 `morestack` 的参数
`divmod` 未知？

```
type m struct {
	// Fields not known to debuggers.
	procid        uint64       // for debuggers, but offset not hard-coded
	gsignal       *g           // signal-handling g
	goSigStack    gsignalStack // Go-allocated signal handling stack
	sigmask       sigset       // storage for saved signal mask
	tls           [6]uintptr   // thread-local storage (for x86 extern register)
	mstartfn      func()
	curg          *g       // current running goroutine
	caughtsig     guintptr // goroutine running during fatal signal
	p             puintptr // attached p for executing go code (nil if not executing go code)
	nextp         puintptr
	oldp          puintptr // the p that was attached before executing a syscall
	id            int64
}
```

`procid` 调度器使用，非硬编码的偏移量, 可以向这个pid发送 signal，参考 [https://blog.haohtml.com/archives/23854](https://blog.haohtml.com/archives/23854)
`gsignal` 信号堆栈 gsignal stack ,作用见[这里][3]
`goSigStack` 分配的信号处理栈，数据类型为 ` [gsignalStack](https://github.com/golang/go/blob/go1.15.6/src/runtime/signal_unix.go#L1168-L1173) `
`sigmask` 存储的信号掩码，数据类型为 `sigset`，作用是？
`tls` 数组类型，本地线程存储，最多为6个
`mstartfn` 表示m启动时立即执行的函数，对其的调用见 ` [mstart1()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L1150-L1180) `
`curg` 当前正在运行的 goroutine
`caughtsig` 在致命信号期间运行的goroutine
`p` 用于执行go code 的 p，就是当前正在m绑定的P，如果没有运行code 的话，值为`nil`
`nextp` 下次运行时的P
`oldp` 在执行系统调用之前绑定的P, 见 [reentersyscall()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L3123) 和 [exitsyscall()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L3243-L3244) 函数
`id` m的唯一id

其中与goroutine有关的字段有`caughtsig` 和 `curg`，其中 `curg` 这个就是当前m绑定的goroutine;

> **gsignal stack 的解释**
>
> 在Golang的runtime包中，gsignal stack（信号堆栈）是用来处理操作系统信号的栈空间。当操作系统发送信号给Go程序时，信号处理函数会在gsignal stack上运行。这个栈是独立于普通的goroutine栈的，用于专门处理信号的相关操作。
>
>
> gsignal stack的主要作用是提供一个独立的执行环境，确保信号处理函数能够正常运行而不受其他goroutine的影响。在处理信号期间，runtime会禁止抢占和栈扩展，以确保信号处理函数的运行不会被干扰。
>
>
> 由于信号处理函数需要尽可能地简洁和高效，gsignal stack的大小是固定的，并且相对较小。这是因为在信号处理期间，只能执行少量的操作，例如发送或接收信号、终止程序等。过多的操作可能会带来不可预知的问题。
>
>
> 需要注意的是，gsignal stack不同于goroutine栈，它是专门用于处理信号的，而goroutine栈则用于正常的程序执行。这样的设计可以有效地隔离信号处理函数和普通程序逻辑，提高信号处理的可靠性和安全性。

与p相关的字段 `p`、`nextp`、`oldp`，分别表示 `当前绑定的P`、`下次绑定的P` 和 `上次绑定的P`，这几个字段均在 `/runtime/proc.go` 文件中使用。

```
type m struct {
	mallocing     int32
	throwing      int32
	preemptoff    string // if != "", keep curg running on this m
	locks         int32
	dying         int32
	profilehz     int32
}
```

`throwing` 当前m抛出异常，即调用了内部函数 `throw`
`preemptoff` 如果非空，则保持curg 运行在当前m。

其它几个字段未知

```
type m struct {
	spinning      bool // m is out of work and is actively looking for work
	blocked       bool // m is blocked on a note
	newSigstack   bool // minit on C thread called sigaltstack
	printlock     int8
	incgo         bool   // m is executing a cgo call
	freeWait      uint32 // if == 0, safe to free g0 and delete m (atomic)
	fastrand      [2]uint32
	needextram    bool
	traceback     uint8
}
```

`spinning` 表示当前m空闲，需要找一个新的工作来执行
`blocked` 在 `note` 阻塞
`newSigstack` 在一个C 线程被调用 sigaltstack
`printlock` ？
`incgo` 当前m正在执行一个cgo调用
`freeWait` 如果值为0,则需要安全的释放go并删除m（原子操作）
`fastrand` ？
`needextram` ?
`traceback` trace 相关

对于 `spinning` 这个情况经常见，当前m没有活干了，需要努力找一个新活干，属于GMP调度中的一个关系点。

note 的数据结构为

```
// sleep and wakeup on one-time events.
// before any calls to notesleep or notewakeup,
// must call noteclear to initialize the Note.
// then, exactly one thread can call notesleep
// and exactly one thread can call notewakeup (once).
// once notewakeup has been called, the notesleep
// will return.  future notesleep will return immediately.
// subsequent noteclear must be called only after
// previous notesleep has returned, e.g. it's disallowed
// to call noteclear straight after notewakeup.
// 一次性事件中的休眠和唤醒
// 对于任何调用 notesleep 或 notwakeup 之前，必须调用 noteclear 进行初始化操作。
// 那么，一个线程调用 notesleep, 一个线程调用 notewakup(只能一次）。
// 当 notewakeup 被调用后，notesleep 还未返回，需要过一段时间 notesleep 才能调用完成。在继续调用 noteclear 之前，必须等待当前一个 notesleep 返回后才可以，
// 不允许在 notewakeup 后直接调用 noteclear。
//
// notetsleep is like notesleep but wakes up after
// a given number of nanoseconds even if the event
// has not yet happened.  if a goroutine uses notetsleep to
// wake up early, it must wait to call noteclear until it
// can be sure that no other goroutine is calling
// notewakeup.
// notetsleep 类似 notesleep, 但唤醒后会返回一个纳秒数值（如果事件正好还不没有发生）
// 如果一个 goroutine 提前使用了 notetsleep 唤醒，它必须等待调用完 noteclear，直到确认没有其它goroutine调用 notewakeup
//
// notesleep/notetsleep are generally called on g0,
// notetsleepg is similar to notetsleep but is called on user g.
// notesleep/notetsleep 通常在 g0 上调用, notetsleepg类似于notetsleep，但在用户g上调用(可能指的用户态的G）

type note struct {
	// Futex-based impl treats it as uint32 key,
	// while sema-based impl as M* waitm.
	// Used to be a union, but unions break precise GC.
	key uintptr
}
```

对于 `note` 这个类型，在调度时经常使用，就有三个与其相关的命令（`notesleep`、`notewakup` 和 `noteclear`, 注意这们的调用顺序要求），参考： [https://www.purewhite.io/2019/11/28/runtime-hacking-translate/#%E5%90%8C%E6%AD%A5](https://www.purewhite.io/2019/11/28/runtime-hacking-translate/#%E5%90%8C%E6%AD%A5)。

```
type m struct {
	ncgocall      uint64      // number of cgo calls in total
	ncgo          int32       // number of cgo calls currently in progress
	cgoCallersUse uint32      // if non-zero, cgoCallers in use temporarily
	cgoCallers    *cgoCallers // cgo traceback if crashing in cgo call
}
```

`ncgocall` cgo的总调用次数
`ncgo` 目前正在执行的cgo调用数
`cgoCallersUse` 如果 > 0，临时使用cgoCallers
`cgoCaller` 如果在调用 cgo 时奔溃，则是进行回溯？

这四个字段主要与cgo相关。

```
type m struct {
	park          note
	alllink       *m // on allm
	schedlink     muintptr
	lockedg       guintptr
	createstack   [32]uintptr // stack that created this thread.
	lockedExt     uint32      // tracking for external LockOSThread
	lockedInt     uint32      // tracking for internal lockOSThread
	nextwaitm     muintptr    // next m waiting for lock
}
```

`park` 数据类型为 `note` , M休眠时使用的信号量, 唤醒M时会通过它唤醒
`alllink` ?
`schedlink` 调度相关,下一个m, 当m在链表结构中会使用
`lockedg` 锁定的goroutine, `g.lockedm`的对应值
`createstatck` 创建当前线程的栈
`lockedExt` 跟踪外部锁线程
`lockedInt` 跟踪内部锁线程
`nextwaitm` 下一个等待此锁的m

```
type m struct {
	waitunlockf   func(*g, unsafe.Pointer) bool
	waitlock      unsafe.Pointer
	waittraceev   byte
	waittraceskip int
	startingtrace bool
	syscalltick   uint32
	freelink      *m // on sched.freem
}
```

`waitunlockf` 等待解锁函数
`waitlock` 等待锁
`waittraceev`
`waittraceskip`
`startingtrace`
`syscalltick`
`freelink` m释放列表，链接到 `sched.freem` 字段

对于 `waitlockf`、 `waitlock` 、`waittraceev`、`waittraceskip` 是与调度相关有关的字段（见 [`gppark()`](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L299-L303) 函数）。`startingtrace` 是与trace相关的三个字段。`freelink` 看注释是与调度有关的。

```
type m strcut {
	// these are here because they are too large to be on the stack
	// of low-level NOSPLIT functions.
	libcall   libcall
	libcallpc uintptr // for cpu profiler
	libcallsp uintptr
	libcallg  guintptr
	syscall   libcall // stores syscall parameters on windows

	vdsoSP uintptr // SP for traceback while in VDSO call (0 if not in call)
	vdsoPC uintptr // PC for traceback while in VDSO call
}
```

前五个字段可能与库调用相关，它们之所以出现在这里，是因为它们太大了，不可能出现在低级NOSPLIT函数的堆栈中。

`vdsoSP` 和 `vdsoPC` 属于`VDSO`调用。vdsoSP 如果值为0，则表示没有调用。

```
type m struct {
	// preemptGen counts the number of completed preemption
	// signals. This is used to detect when a preemption is
	// requested, but fails. Accessed atomically.
	preemptGen uint32

	// Whether this is a pending preemption signal on this M.
	// Accessed atomically.
	signalPending uint32

	dlogPerM

	mOS

	// Up to 10 locks held by this m, maintained by the lock ranking code.
	locksHeldLen int
	locksHeld    [10]heldLockInfo
}
```

`preemptGen`完成的抢占信号数量。这用于检测何时请求抢占，但是失败了。原子访问
`signalPending` 是不是当前M上挂起的抢占信号
`dlogPerM` ?
`mOS` ?
`locksHeldLen` 当前持有锁的数量
`locksHeld` 最多可以持有1个锁，数据类型为 `heldLockInfo`,

`preemptGen` 和 `signalPending` 是与抢占有关的字段；
`locksHeldLen` 和 `locksHeld` 是表示持有锁的信息，他们由`ranking code` 维护。

对于 `heldLockInfo` 数据结构

```
// heldLockInfo gives info on a held lock and the rank of that lock
// heldLockInfo 提供了一个锁的相关信息和锁的等级
type heldLockInfo struct {
	lockAddr uintptr
	rank     lockRank
}
```

```
// src/runtime/lockrank.go

type lockRank int
```

**总结**

 * 每个 m 都持有一个特殊的goroutine，我们称之为g0，它具有调度栈的能力。调度和执行系统调用时会先切换到这个g
 * m 可以与一个G相绑定，字段 `curg`
 * m 可以与P绑定，也可以不绑定，如果已绑定则字段p不非`nil`
 * m 可以记录下次和上次绑定的p，字段 `nextp` 和 `oldp`
 * 如果m 没有工作可做的话，它会通过自旋积极再找一个活干
 * m会不会在 `note` 阻塞
 * 可以记录对m完成抢占信息的次数
 * 一个m最多可以持有10个锁

# Sched 

上面我们分别介绍了 G/P/M 三者的数据结构，想必对他们结构体中的每个字段的作用有了一点的了解了，然而三者的协作需要一个负责协调的角色，而这正是 `go sched` 的职责。

**Go 调度器，**它维护有存储 M 和 G 的队列以及调度器的一些状态信息等，全局调度时使用。

调度器循环的机制大致是从各种队列、P 的本地队列中获取 G，然后切换到 G 的执行栈上并执行 G 的函数，调用 Goexit 做清理工作并回到 M，如此反复。

对于三者调度逻辑可参考文章 [Go netpoller 网络模型之源码全面解析](https://mp.weixin.qq.com/s/HNPeffn08QovQwtUH1qkbQ)

下面我们再看一下它的数据结构

```
type schedt struct {
	// accessed atomically. keep at top to ensure alignment on 32-bit systems.
	// 原子访问, 最顶部，保证32位系统下的对齐
	goidgen uint64

	// 上次网络轮询的时间,如果当前正在轮询,则为0
	lastpoll uint64 // time of last network poll, 0 if currently polling

	// 当前轮询休眠的时间
	pollUntil uint64 // time to which current poll is sleeping

	lock mutex

	// When increasing nmidle, nmidlelocked, nmsys, or nmfreed, be
	// sure to call checkdead().
	// 当增加 nmidle，nmidlelocked, nmsys或 nmfreed 的时候，一定要调用 checkdead()

	// 空闲m等待队列
	midle muintptr // idle m's waiting for work

	// 空闲m的数量
	nmidle int32 // number of idle m's waiting for work

	// 等待工作的锁定m的数量
	nmidlelocked int32 // number of locked m's waiting for work

	// 已创建的m数和下一个m ID, 一个字段代表两个意义？
	mnext int64 // number of m's that have been created and next M ID

	// 允许的最大m数
	maxmcount int32 // maximum number of m's allowed (or die)

	// 死锁不计算系统m的数量
	nmsys int32 // number of system m's not counted for deadlock

	// 累计已释放m的数量
	nmfreed int64 // cumulative number of freed m's

	// 系统goroutins的数量，原子更新
	ngsys uint32 // number of system goroutines; updated atomically

	// 空闲p
	pidle  puintptr // idle p's
	npidle uint32
	// 自旋, 查看proc.go 文件的 "Worker thread parking/unparking" 注释
	nmspinning uint32 // See "Worker thread parking/unparking" comment in proc.go.

	// Global runnable queue.
	// 全局运行队列信息, gQueue 是一个通过 g.schedlink 链接的双向队列
	runq     gQueue
	// 队列大小
	runqsize int32

	// disable controls selective disabling of the scheduler.
	//
	// Use schedEnableUser to control this.
	//
	// disable is protected by sched.lock.
	disable struct {
		// user disables scheduling of user goroutines.
		user     bool
		runnable gQueue // pending runnable Gs
		n        int32  // length of runnable
	}

	// Global cache of dead G's.
	gFree struct {
		lock    mutex
		stack   gList // Gs with stacks
		noStack gList // Gs without stacks
		n       int32
	}

	// Central cache of sudog structs.
	sudoglock  mutex
	sudogcache *sudog

	// Central pool of available defer structs of different sizes.
	deferlock mutex
	deferpool [5]*_defer

	// freem is the list of m's waiting to be freed when their
	// m.exited is set. Linked through m.freelink.
	// freem 是当 他们的 m.exited 被设置时的等待被释放m列表，通过 m.freelink 链接
	freem *m

	// gc正在等待运行
	gcwaiting  uint32 // gc is waiting to run
	stopwait   int32
	stopnote   note
	sysmonwait uint32
	sysmonnote note

	// safepointFn should be called on each P at the next GC
	// safepoint if p.runSafePointFn is set.
	// 如果 p.runSafePointFn 设置的话, safeopintFn 将在每个p下次 GC safepoint 时被调用
	safePointFn   func(*p)
	safePointWait int32
	safePointNote note

	profilehz int32 // cpu profiling rate

	// 对gomaxprocs的最后更改时间
	procresizetime int64 // nanotime() of last change to gomaxprocs
	totaltime      int64 // ∫gomaxprocs dt up to procresizetime

	// sysmonlock protects sysmon's actions on the runtime.
	//
	// Acquire and hold this mutex to block sysmon from interacting
	// with the rest of the runtime.
	// 在运行时，sysmonlock保护 sysmon 的运行
	sysmonlock mutex
}
```

```
type schedt struct {
	// accessed atomically. keep at top to ensure alignment on 32-bit systems.
	// 原子访问, 最顶部位置，保证32位系统下的对齐
	goidgen uint64

	// 上次网络轮询的时间,如果当前正在轮询,则为0
	lastpoll uint64 // time of last network poll, 0 if currently polling

	// 当前轮询休眠的时间
	pollUntil uint64 // time to which current poll is sleeping

	lock mutex
}
```

`goidgen` 目前还不清楚它的作用
`lastpoll` 上次网络轮询的时间，如果值为0，表示当前正在轮询
`pollUntil` 当前由于轮询导致的休眠的时间
`lock` 锁

```
type schedt struct {
	// When increasing nmidle, nmidlelocked, nmsys, or nmfreed, be
	// sure to call checkdead().
	// 当增加 nmidle，nmidlelocked, nmsys或 nmfreed 的时候，一定要调用 checkdead() 函数

	// 空闲m等待时间
	midle muintptr // idle m's waiting for work

	// 空闲m的数量
	nmidle int32 // number of idle m's waiting for work

	// 等待工作的锁定m的数量
	nmidlelocked int32 // number of locked m's waiting for work

	// 已创建的m数和下一个m ID, 一个字段代表两个意义？
	mnext int64 // number of m's that have been created and next M ID

	// 允许的最大m数
	maxmcount int32 // maximum number of m's allowed (or die)

	// 死锁不计算系统m的数量
	nmsys int32 // number of system m's not counted for deadlock

	// 累计已释放m的数量
	nmfreed int64 // cumulative number of freed m's

	// 系统goroutins的数量，原子更新
	ngsys uint32 // number of system goroutines; updated atomically
}
```

`midle` 等待工作的空闲m
`nmidle` 空闲m的数量
`nmidlelocked` 空间等待m中，加锁的数量
`mnext` 已创建的m数量和下一个m ID, 一个字段代表两个意义？（如果为9的话，则下个使用的m 就是9号m吗？）
`maxmcount` 允许使用的最大m数量（空闲m）
`nmsys` 由于死锁未被计算在内的m的数量
`nmfreed` 累计已释放m的数量
`ngsys` 系统groutines的数量，原子更新

调度器会记录当前哪个m在等待工作(`midle`)，一共有多少个空闲m(`nmidle`)，同时还记录等待工作的锁定m的数量(`nmidlelocked`), 已创建m的数量和允许的最大空间m数量(`maxmcount`)等信息

```
type schedt struct {
	// 空闲p
	pidle  puintptr // idle p's
	npidle uint32
	// 自旋, 查看proc.go 文件的 "Worker thread parking/unparking" 注释
	nmspinning uint32 // See "Worker thread parking/unparking" comment in proc.go.

	// Global runnable queue.
	// 全局运行队列信息, gQueue 是一个通过 g.schedlink 链接的双向队列
	runq     gQueue
	// 队列大小
	runqsize int32
}
```

`pidle` 空间的P
`npidle` 空闲P的数量
`nmspinning` 正在自旋的m的数量，查看proc.go 文件的 “Worker thread parking/unparking” 注释
`runq` 全局G队列（还记得吧，P也有自己的G队列，最大值为256）
`runqsize` 全局队列大小

记录了当前空闲的P是哪一个(`pidle`)，共有几少个空闲的P(`npidle`)，还有多少个正在spinning 的M(`nmspinning`)。另外还有一个全局G的列队（`runq`）。是不是越来越有点意思了，有了空闲的M（`midle`）及其数量(`nmidle`)，也有了空闲的P(`pidle`)及其数量(`npidle`)，还知道有多少个M正在在spinning（`nmspinning`）拼命在找活干，除了每个P下面自己的runq队列，调度器自身也有存放G的队列`runq`，所需要的资源GPM都够了，剩下的就是看如何给他们近排活干了。

```
type schedt struct {
	// disable controls selective disabling of the scheduler.
	//
	// Use schedEnableUser to control this.
	//
	// disable is protected by sched.lock.
	disable struct {
		// user disables scheduling of user goroutines.
		user     bool
		runnable gQueue // pending runnable Gs
		n        int32  // length of runnable
	}

	// Global cache of dead G's.
	gFree struct {
		lock    mutex
		stack   gList // Gs with stacks
		noStack gList // Gs without stacks
		n       int32
	}
}
```

`disable` 禁用受保护调度锁 `sched.lock`，匿名结构体。
`gFree` 全局缓存G， 匿名结构体。如果一个G完毕后，并不立即释放它而是先放在一个列表里，以备后续复用，这样可以优先减少创建资源的开销。放入g后再检查一下当前 `p` 的 `p.gFree` 长度是否>=64（`_p_.gFree.n >= 64`） , 如果大于则会将 `p.gFree` 一半的 `g` 迁移到 `sched.gFree`，源码见 ` [gfput()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L3710-L3743) ` 。

对于 `disable` 字段的值受 `schedEnableUser` 控制， 可以用来控制用户goroutines 的调度。

```
type schedt struct {
	// Central cache of sudog structs.
	sudoglock  mutex
	sudogcache *sudog

	// Central pool of available defer structs of different sizes.
	deferlock mutex
	deferpool [5]*_defer

	// freem is the list of m's waiting to be freed when their
	// m.exited is set. Linked through m.freelink.
	// freem 是当 他们的 m.exited 被设置时的等待被释放m列表，通过 m.freelink 链接
	freem *m
}
```

`sudoglock` sudog锁
`sudogcache` sudog结构的中央缓存
`deferlock` defer锁
`deferpool` 数组，最大值5，不同大小的defer pool 池
`freem` 它`m.exited` 设置的时候，freem就是等待被释放的列表，通过 m.freelink 链接

一个sudog缓存，另一个是defer pool，各自都有一个对应的锁。

```
type schedt strcut {
	// gc正在等待运行
	gcwaiting  uint32 // gc is waiting to run
	stopwait   int32
	stopnote   note
	sysmonwait uint32
	sysmonnote note
}
```

这几个都是与等待有关的字段，`gcwaiting` 与GC有关，xxxnote的与 note 有关。

```
type schedt strcut {
	// safepointFn should be called on each P at the next GC
	// safepoint if p.runSafePointFn is set.
	// 如果 p.runSafePointFn 设置的话, safeopintFn 将在每个p下次 GC safepoint 时被调用
	safePointFn   func(*p)
	safePointWait int32
	safePointNote note

	profilehz int32 // cpu profiling rate

	// 对gomaxprocs的最后更改时间
	procresizetime int64 // nanotime() of last change to gomaxprocs
	totaltime      int64 // ∫gomaxprocs dt up to procresizetime

	// sysmonlock protects sysmon's actions on the runtime.
	//
	// Acquire and hold this mutex to block sysmon from interacting
	// with the rest of the runtime.
	// 在运行时，sysmonlock保护 sysmon 的运行
	sysmonlock mutex
}
```

前三个是与安全点有关的字段，目前还不清楚SafePoint 指的是什么？
`sysmonlock` 是sysmon协程锁![](https://blogstatic.haohtml.com/uploads/2021/01/fa82a74a3feb15ed2757ded042dc984d.jpg)sysmon 协程

**总结**

 * 调度器会记录当前是否处于轮训状态以及轮训的时间
 * 记录有M相关的信息，如当前空闲的M，如果有多个的话，也会记录个数，记录的还有已用过数量，当前有多少个正在spinning，最大允许的M数量。同时也会记录其中持有锁的数量
 * 记录有P相关的信息，如当前空闲P，空闲的数量。
 * 持有当前系统 groutine 数量
 * 有一个G的全局运行队列及其队列大小
 * 通过gFree 记录有多少个空闲的G，可以被重复利用
 * sudog缓存 和 deferpool
 * 都有一个全局锁（lock）和 sysmon (sysmonlock)锁及其它锁(sugoglock)
 * 可以控制是否禁用用户gorutine的调度行为,字段 disable(调用 schedEnableUser)

如果想学习Golang 的内存管理，则推荐先看这一篇 [https://blog.haohtml.com/archives/29385](https://blog.haohtml.com/archives/29385)

# 参考 

 * [https://www.purewhite.io/2019/11/28/runtime-hacking-translate/](https://www.purewhite.io/2019/11/28/runtime-hacking-translate/)
 * [https://www.zhihu.com/question/20862617/answer/27964865](https://www.zhihu.com/question/20862617/answer/27964865)
 * [https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/)
 * [https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/)
 * [Go：g0，特殊的 Goroutine](https://studygolang.com/articles/28443)
 * [GPM 是什么](https://github.com/golang-design/Go-Questions/blob/master/goroutine%20%E8%B0%83%E5%BA%A6%E5%99%A8/GPM%20%E6%98%AF%E4%BB%80%E4%B9%88.md)
 * [Go 为什么这么“快”](https://mp.weixin.qq.com/s/ihJFa5Wir4ohhZUXVSBvMQ)
 * [Golang和假共享(false sharing)](https://www.xwxwgo.com/post/2019/07/09/golang%E5%92%8Cfalse-sharing/)
 * [CPU缓存体系对Go程序的影响][4]
 * [锁、内存屏障与缓存一致性][5]
 * [理解 Go 并发以及调度器亲和性](https://mp.weixin.qq.com/s/XnqF5aZ_0F3-cZPUm9pM7w)
 * [https://www.zhihu.com/question/20862617](https://www.zhihu.com/question/20862617)
 * [https://blog.csdn.net/u010853261/article/details/84790392](https://blog.csdn.net/u010853261/article/details/84790392)
 * [https://medium.com/a-journey-with-go/go-memory-management-and-allocation-a7396d430f44](https://medium.com/a-journey-with-go/go-memory-management-and-allocation-a7396d430f44)
 * [Go netpoller 网络模型之源码全面解析](https://mp.weixin.qq.com/s/HNPeffn08QovQwtUH1qkbQ)
 * [https://studygolang.com/articles/11627](https://studygolang.com/articles/11627)
 * [StackGuard的作用][6]

 [1]: https://draveness.me/golang/tree/runtime.mallocgc
 [2]: https://github.com/golang/go/blob/go1.15.6/src/runtime/stack.go#L943
 [3]: https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L1237-L1245
 [4]: https://mp.weixin.qq.com/s/vnm9yztpfYA4w-IM6XqyIA
 [5]: https://gocode.cc/project/9/article/128
 [6]: https://jiajunhuang.com/articles/2020_08_26-stackguard.md.html