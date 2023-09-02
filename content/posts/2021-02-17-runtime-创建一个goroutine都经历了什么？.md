---
title: 'Runtime: 创建一个goroutine都经历了什么？'
author: admin
type: post
date: 2021-02-17T04:22:00+00:00
url: /archives/23168
categories:
 - 程序开发
tags:
 - golang
 - runtime

---
我们都知道goroutine的在golang中发挥了很大的作用，那么当我们创建一个新的goroutine时，它是怎么一步一步创建的呢？都经历了哪些操作呢？今天我们通过源码来剖析一下创建goroutine都经历了些什么？go version 1.15.6

对goroutine最关键的两个函数是 ` [newproc()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L3535-L3564) ` 和 ` [newproc1()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L3566-L3674) `，而 `newproc1()` 函数是我们最需要关注的。

## 函数 newproc() 

我们先看一个简单的创建goroutine的例子，找出来创建它的函数。

```
package main

func start(a, b, c int64) {
	_ = a + b + c
}

func main() {
	go start(7, 2, 5)
}

```

输出结果:

```
➜  gotest go tool compile -S main.go
"".start STEXT nosplit size=1 args=0x18 locals=0x0
        0x0000 00000 (main.go:3)        TEXT    "".start(SB), NOSPLIT|ABIInternal, $0-24
        0x0000 00000 (main.go:3)        FUNCDATA        $0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
        0x0000 00000 (main.go:3)        FUNCDATA        $1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
        0x0000 00000 (main.go:5)        RET
        0x0000 c3                                               .
"".main STEXT size=98 args=0x0 locals=0x30
        0x0000 00000 (main.go:7)        TEXT    "".main(SB), ABIInternal, $48-0
        0x0000 00000 (main.go:7)        MOVQ    (TLS), CX
        0x0009 00009 (main.go:7)        CMPQ    SP, 16(CX)
        0x000d 00013 (main.go:7)        PCDATA  $0, $-2
        0x000d 00013 (main.go:7)        JLS     90
        0x000f 00015 (main.go:7)        PCDATA  $0, $-1
        0x000f 00015 (main.go:7)        SUBQ    $48, SP
        0x0013 00019 (main.go:7)        MOVQ    BP, 40(SP)
        0x0018 00024 (main.go:7)        LEAQ    40(SP), BP
        0x001d 00029 (main.go:7)        FUNCDATA        $0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
        0x001d 00029 (main.go:7)        FUNCDATA        $1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
        0x001d 00029 (main.go:8)        MOVL    $24, (SP)
        0x0024 00036 (main.go:8)        LEAQ    "".start·f(SB), AX
        0x002b 00043 (main.go:8)        MOVQ    AX, 8(SP)
        0x0030 00048 (main.go:8)        MOVQ    $7, 16(SP)
        0x0039 00057 (main.go:8)        MOVQ    $2, 24(SP)
        0x0042 00066 (main.go:8)        MOVQ    $5, 32(SP)
        0x004b 00075 (main.go:8)        PCDATA  $1, $0
        0x004b 00075 (main.go:8)        CALL    runtime.newproc(SB)
        0x0050 00080 (main.go:9)        MOVQ    40(SP), BP
        0x0055 00085 (main.go:9)        ADDQ    $48, SP
        0x0059 00089 (main.go:9)        RET
        0x005a 00090 (main.go:9)        NOP
        0x005a 00090 (main.go:7)        PCDATA  $1, $-1
        0x005a 00090 (main.go:7)        PCDATA  $0, $-2
        0x005a 00090 (main.go:7)        CALL    runtime.morestack_noctxt(SB)
        0x005f 00095 (main.go:7)        PCDATA  $0, $-1
        0x005f 00095 (main.go:7)        NOP
        0x0060 00096 (main.go:7)        JMP     0
        0x0000 65 48 8b 0c 25 00 00 00 00 48 3b 61 10 76 4b 48  eH..%....H;a.vKH
        0x0010 83 ec 30 48 89 6c 24 28 48 8d 6c 24 28 c7 04 24  ..0H.l$(H.l$(..$
        0x0020 18 00 00 00 48 8d 05 00 00 00 00 48 89 44 24 08  ....H......H.D$.
        0x0030 48 c7 44 24 10 07 00 00 00 48 c7 44 24 18 02 00  H.D$.....H.D$...
        0x0040 00 00 48 c7 44 24 20 05 00 00 00 e8 00 00 00 00  ..H.D$ .........
        0x0050 48 8b 6c 24 28 48 83 c4 30 c3 e8 00 00 00 00 90  H.l$(H..0.......
        0x0060 eb 9e                                            ..
        rel 5+4 t=17 TLS+0
        rel 39+4 t=16 "".start·f+0
        rel 76+4 t=8 runtime.newproc+0
        rel 91+4 t=8 runtime.morestack_noctxt+0
go.cuinfo.packagename. SDWARFINFO dupok size=0
        0x0000 6d 61 69 6e                                      main
""..inittask SNOPTRDATA size=24
        0x0000 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
        0x0010 00 00 00 00 00 00 00 00                          ........
"".start·f SRODATA dupok size=8
        0x0000 00 00 00 00 00 00 00 00                          ........
        rel 0+8 t=1 "".start+0
gclocals·33cdeccccebe80329f1fdbee7f5874cb SRODATA dupok size=8
        0x0000 01 00 00 00 00 00 00 00                          ........
```

通过使用 `go tool compile -S main.go` 命令输出的汇编我们基本可以看到有一个 ` [runtime.newproc()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L3535-L3564) ` 函数的调用（`CALL` 是汇编调用函数指令） ，这个正是`runtime`创建`goroutine`的关键函数。

我们先看一个官方对`newproc()` 函数的注释说明。

```
// Create a new g running fn with siz bytes of arguments.
// Put it on the queue of g's waiting to run.
// The compiler turns a go statement into a call to this.
//
// The stack layout of this call is unusual: it assumes that the
// arguments to pass to fn are on the stack sequentially immediately
// after &fn. Hence, they are logically part of newproc's argument
// frame, even though they don't appear in its signature (and can't
// because their types differ between call sites).
//
// This must be nosplit because this stack layout means there are
// untyped arguments in newproc's argument frame. Stack copies won't
// be able to adjust them and stack splits won't be able to copy them.
//
//go:nosplit
func newproc(siz int32, fn *funcval) {}
```

通过官方注释，可得知以下信息：

 1. 创建一个`goroutine`，是通过调用一个 `fn` 函数来实现的，调用时需要指定`参数大小`
 2. 创建完后需要将goroutine放在一个运行队列（P的本地队列或全局队列）
 3. 编译器通过一个`go关键字` 实现对函数的调用
 4. stack layout 是特殊的
 5. 函数是`nosplit` ，不允许分裂的

`funcval` 是一个结构体类型

```
type funcval struct {
	fn uintptr
	// variable-size, fn-specific data here
}

```

```
//go:nosplit
func newproc(siz int32, fn *funcval) {
	// 获取参数，堆栈分配顺序是从上（高）到下（低）增长
	argp := add(unsafe.Pointer(&fn), sys.PtrSize)

	// 获取当前g,初始化时为 m0.g0
	gp := getg()

	// 获取caller PC
	pc := getcallerpc()

	// 在 系统栈上运行（切换到g0执行）
	systemstack(func() {
		// 创建一个g, 关注重点！重点！重点！
		newg := newproc1(fn, argp, siz, gp, pc)

		_p_ := getg().m.p.ptr()
		// 调用 函数runqput() 将newg 放在P的运行队列
		// 第三个参数true表示将这个newg放在p.runnext字段(亲合性考虑），如果p.runnext已存在，则需要将oldg替换掉，并将其放在全局运行队列，false 表示放在全局运行队列。
		// 如果P已满，则放在全局运行队列
		runqput(_p_, newg, true)

		if mainStarted {
			wakep()
		}
	})
}
```

可以看出newproc()函数只是对newproc1() 函数的一个封装。

` [getcallerpc()](https://github.com/golang/go/blob/go1.15.6/src/runtime/stubs.go#L212-L237) `函数用来获取调用者的`program counter(PC)` ，最终会转换为汇编实现调用，类似的还有`getcallersp()`。官方注释如下

```
// getcallerpc returns the program counter (PC) of its caller's caller.
// getcallersp returns the stack pointer (SP) of its caller's caller.
// The implementation may be a compiler intrinsic; there is not
// necessarily code implementing this on every platform.
//
// For example:
//
//	func f(arg1, arg2, arg3 int) {
//		pc := getcallerpc()
//		sp := getcallersp()
//	}
//
// These two lines find the PC and SP immediately following
// the call to f (where f will return).
//
// The call to getcallerpc and getcallersp must be done in the
// frame being asked about.
//
// The result of getcallersp is correct at the time of the return,
// but it may be invalidated by any subsequent call to a function
// that might relocate the stack in order to grow or shrink it.
// A general rule is that the result of getcallersp should be used
// immediately and can only be passed to nosplit functions.

//go:noescape
func getcallerpc() uintptr

//go:noescape
func getcallersp() uintptr // implemented as an intrinsic on all platforms
```

`getcallersp()` 函数调用时结果是正确的，但随着随后堆栈的扩容和缩容结果很可能是错误的。一般规则是将 `getcallersp()` 的结果立即传递给`nosplit`函数，正如这里 `newproc()` 函数对 `newproc1()` 的调用方法一样。

对`PC寄存器`的介绍参考 [这里](https://www.zhihu.com/question/22609253)。

## 函数newproc1() 

我们还是先看一下函数注释。

```
// Create a new g in state _Grunnable, starting at fn, with narg bytes
// of arguments starting at argp. callerpc is the address of the go
// statement that created this. The caller is responsible for adding
// the new g to the scheduler.
//
// This must run on the system stack because it's the continuation of
// newproc, which cannot split the stack.
//
//go:systemstack
func newproc1(fn *funcval, argp unsafe.Pointer, narg int32, callergp *g, callerpc uintptr) *g {}
```

创建一个G(状态为 `_Grunnable`)，从 fn 开始。其参数大小为 nargs 字节，从argp开始。
callerpc 是创建go 语句的地址，caller负责将新创建的g发送调度程序。

函数必须运行在系统栈上（`go:systemstack`)，不允许 split，它和函数 `newproc()` 是连续一起的。

**参数说明**
`fn`: 要执行的函数
`argp`: 函数的第一个参数地址
`narg`: 参数总字节大小
`callergp`: 这里是g0
`callerpc`: caller PC, 创建go语句的地址

现在我们再看一下函数体内容

```
//go:systemstack
func newproc1(fn *funcval, argp unsafe.Pointer, narg int32, callergp *g, callerpc uintptr) *g {
	// 1. 获取一个g（在本文中当前已处在系统栈，所以获取的其实还是m0.g0)
	_g_ := getg()

	if fn == nil {
		_g_.m.throwing = -1 // do not dump full stacks
		throw("go of nil func value")
	}

	// 2. 对m.locks+1，对应函数尾的releasem(_g_.m)
	acquirem() // disable preemption because it can be holding p in a local var
	siz := narg
	siz = (siz + 7) &^ 7

	// We could allocate a larger initial stack if necessary.
	// Not worth it: this is almost always an error.
	// 4*sizeof(uintreg): extra space added below
	// sizeof(uintreg): caller's LR (arm) or return address (x86, in gostartcall).
	if siz >= _StackMin-4*sys.RegSize-sys.RegSize {
		throw("newproc: function arguments too large for new goroutine")
	}

	// 3. 从当前P中获取一个空闲g（p.gFree队列）进行复用,如果p的g为空，则从全局运行队列窃取一些,默认 g 栈大小为2K
	// 重点！重点！如果从队列里获取的g的stack为0，则会重复申请 _FixedStack 大小的stack,见 https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L3773-L3786，至于为什么可见：https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L3719-L3725
	_p_ := _g_.m.p.ptr()
	newg := gfget(_p_)

	// 3.1 找不到空闲g，则创建一个新的g
	if newg == nil {
		newg = malg(_StackMin)
		// 修改刚创建的g状态为 _Gdead,表示刚刚初始化，gc扫描器不会搜索这类状态的堆栈
		casgstatus(newg, _Gidle, _Gdead)
		// 将g放在allgs切片变量里
		allgadd(newg) // publishes with a g->status of Gdead so GC scanner doesn't look at uninitialized stack.
	}
	if newg.stack.hi == 0 {
		throw("newproc1: newg missing stack")
	}
	// 当前g 状态必须是 _Gdead
	if readgstatus(newg) != _Gdead {
		throw("newproc1: new g is not Gdead")
	}

	// g堆栈大小
	totalSize := 4*sys.RegSize + uintptr(siz) + sys.MinFrameSize // extra space in case of reads slightly beyond frame
	totalSize += -totalSize & (sys.SpAlign - 1)                  // align to spAlign
	sp := newg.stack.hi - totalSize
	spArg := sp
	if usesLR {
		// caller's LR
		*(*uintptr)(unsafe.Pointer(sp)) = 0
		prepGoExitFrame(sp)
		spArg += sys.MinFrameSize
	}

	// 4. fn参数大小(将参数内容从g0堆栈复制到newg的堆栈)
	if narg > 0 {
		// 将参数堆栈（g0栈）内容从复制到newg（新g）的堆栈
		memmove(unsafe.Pointer(spArg), argp, uintptr(narg))
		// This is a stack-to-stack copy. If write barriers
		// are enabled and the source stack is grey (the
		// destination is always black), then perform a
		// barrier copy. We do this *after* the memmove
		// because the destination stack may have garbage on
		// it.
		if writeBarrier.needed && !_g_.m.curg.gcscandone {
			f := findfunc(fn.fn)
			stkmap := (*stackmap)(funcdata(f, _FUNCDATA_ArgsPointerMaps))
			if stkmap.nbit > 0 {
				// We're in the prologue, so it's always stack map index 0.
				bv := stackmapdata(stkmap, 0)
				bulkBarrierBitmap(spArg, spArg, uintptr(bv.n)*sys.PtrSize, 0, bv.bytedata)
			}
		}
	}

	memclrNoHeapPointers(unsafe.Pointer(&newg.sched), unsafe.Sizeof(newg.sched))

	// 5. 初始化 newg 调度相关字段，调度器需要这些信息才可以将g调度到不同的 p去执行
	newg.sched.sp = sp
	newg.stktopsp = sp

	// 5.1 把返回地址复制到g的栈上, 这里的返回地址是goexit, 表示调用完目标函数后会调用goexit
	newg.sched.pc = funcPC(goexit) + sys.PCQuantum // +PCQuantum so that previous instruction is in same function
	newg.sched.g = guintptr(unsafe.Pointer(newg)) // 设置sched.g = newg

	// 5.2 调度字段 调整gobuf信息 gostartcallfn() -> gostartcall()
	gostartcallfn(&newg.sched, fn)

	// 5.3 创建此goroutine的go语句的pc
	newg.gopc = callerpc

	newg.ancestors = saveAncestors(callergp)

	// 5.4 goroutine函数的pc
	newg.startpc = fn.fn

	if _g_.m.curg != nil {
		newg.labels = _g_.m.curg.labels
	}
	if isSystemGoroutine(newg, false) {
		atomic.Xadd(&sched.ngsys, +1)
	}

	// 6. 设置g状态为 _Grunnable, 让P准备执行
	casgstatus(newg, _Gdead, _Grunnable)

	// 7. 缓存goid
	if _p_.goidcache == _p_.goidcacheend {
		// Sched.goidgen is the last allocated id,
		// this batch must be [sched.goidgen+1, sched.goidgen+GoidCacheBatch].
		// At startup sched.goidgen=0, so main goroutine receives goid=1.
		_p_.goidcache = atomic.Xadd64(&sched.goidgen, _GoidCacheBatch)
		_p_.goidcache -= _GoidCacheBatch - 1
		_p_.goidcacheend = _p_.goidcache + _GoidCacheBatch
	}
	newg.goid = int64(_p_.goidcache)
	_p_.goidcache++
	if raceenabled {
		newg.racectx = racegostart(callerpc)
	}
	if trace.enabled {
		traceGoCreate(newg, newg.startpc)
	}

	// 8. 释放m,其实是减少对m的lock数量，这里涉及到g.stackguard0字段保护,对应上面的 acquirem()
	releasem(_g_.m)

	return newg
}
```

当首次创建 `g` 会通过 ` [malg()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L3518-L3533) ` 函数分配一个具有最小栈的goroutine.

至此，goroutine已经创建成功并处于`_Grunnable` 状态，对于它的执行只需要有个P来执行就可以了。

这里再提醒一次，`newproc1()` 函数是运行在系统栈上，栈地址和 `newproc()` 是连续的。

对于字段 `newg.sched.pc` 值为赋值 `goexit()` 函数+1, 请参考 [这里](https://www.cnblogs.com/abozhang/p/10825342.html)

这里重点理解下这几个字段的作用

`newg.sched.sp

newg.sched.pc

newg.gopc

newg.startpc`

理解SP PC 这些概念时，需要知道栈的布局 [Stack frame layout](https://github.com/golang/go/blob/release-branch.go1.15/src/runtime/stack.go#L505)。

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
```

## 参考资料 

 * [https://golang.design/under-the-hood//zh-cn/part2runtime/ch06sched/stack/](https://golang.design/under-the-hood//zh-cn/part2runtime/ch06sched/stack/)
 * [腾讯：汇编是深入理解 Go 的基础](https://mp.weixin.qq.com/s/2JQM1piaWPQW-uwD_P-3Cg)
 * [https://studygolang.com/articles/11627](https://studygolang.com/articles/11627)
 * [https://www.cnblogs.com/abozhang/p/10825342.html](https://www.cnblogs.com/abozhang/p/10825342.html)
 * [https://zhuanlan.zhihu.com/p/56750445](https://zhuanlan.zhihu.com/p/56750445)
 * [https://www.zhihu.com/question/22609253](https://www.zhihu.com/question/22609253)
 * [https://zh.wikipedia.org/wiki/程式计数器](https://zh.wikipedia.org/wiki/%E7%A8%8B%E5%BC%8F%E8%A8%88%E6%95%B8%E5%99%A8)