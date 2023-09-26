---
title: 'Runtime: goroutine的暂停和恢复源码剖析'
author: admin
type: post
date: 2021-05-10T05:50:57+00:00
url: /archives/30519
toc: true
categories:
 - 程序开发
tags:
 - golang
 - goroutine

---
上一节《 [GC 对根对象扫描实现的源码分析](https://blog.haohtml.com/archives/27003)》中，我们提到过在GC的时候，在对一些goroutine 栈进行扫描时，会在其扫描前触发 G 的暂停(` [suspendG](https://github.com/golang/go/blob/go1.16.2/src/runtime/preempt.go#L76-L254) `)和恢复(` [resumeG](https://github.com/golang/go/blob/go1.16.2/src/runtime/preempt.go#L256-L280) `)。

```
// markroot scans the i'th root.
//
// Preemption must be disabled (because this uses a gcWork).
//
// nowritebarrier is only advisory here.
//
//go:nowritebarrier
func markroot(gcw *gcWork, i uint32) {
	baseFlushCache := uint32(fixedRootCount)
	baseData := baseFlushCache + uint32(work.nFlushCacheRoots)
	baseBSS := baseData + uint32(work.nDataRoots)
	baseSpans := baseBSS + uint32(work.nBSSRoots)
	baseStacks := baseSpans + uint32(work.nSpanRoots)
	end := baseStacks + uint32(work.nStackRoots)

	// Note: if you add a case here, please also update heapdump.go:dumproots.
	switch {
		......

	default:
		var gp *g
		if baseStacks <= i && i < end {
			gp = allgs[i-baseStacks]
		} else {
			throw("markroot: bad index")
		}

		status := readgstatus(gp) // We are not in a scan state
		if (status == _Gwaiting || status == _Gsyscall) && gp.waitsince == 0 {
			gp.waitsince = work.tstart
		}

		// scanstack must be done on the system stack in case
		// we're trying to scan our own stack.
		systemstack(func() {
			userG := getg().m.curg
			selfScan := gp == userG && readgstatus(userG) == _Grunning
			if selfScan {
				casgstatus(userG, _Grunning, _Gwaiting)
				userG.waitreason = waitReasonGarbageCollectionScan
			}

			// TODO: suspendG blocks (and spins) until gp
			// stops, which may take a while for
			// running goroutines. Consider doing this in
			// two phases where the first is non-blocking:
			// we scan the stacks we can and ask running
			// goroutines to scan themselves; and the
			// second blocks.
			stopped := suspendG(gp)
			if stopped.dead {
				gp.gcscandone = true
				return
			}
			if gp.gcscandone {
				throw("g already scanned")
			}
			scanstack(gp, gcw)
			gp.gcscandone = true
			resumeG(stopped)

			if selfScan {
				casgstatus(userG, _Gwaiting, _Grunning)
			}
		})
	}
}
```

那么它在暂停和恢复一个goroutine时都做了些什么工作呢，今天我们通过源码来详细看一下。 go version 1.16.2

## safe point 

首先，我们了解一下golang中的安全点 safe point .

在Golang中，safe point（安全点）是指程序执行时的一个安全点，也可以理解为一种 `同步点`，用于确保垃圾回收（GC）和其他系统操作可以安全地执行。

在Golang中，垃圾回收器是通过停止-复制（stop-the-world）的方式实现的。当垃圾回收器运行时，它需要停止程序的执行，扫描堆内存来识别和回收不再使用的对象。然而，在停止期间，程序的所有goroutines都处于暂停状态。为了减少停止的时间，Golang在适当的地方插入了安全点。

在Golang中，安全点的位置是在函数调用、循环迭代、系统调用等等需要长时间运行的操作之后。当goroutine到达安全点时，垃圾回收器可以安全地挂起该goroutine，并扫描其中的对象。一旦垃圾回收完成，goroutine会被重新唤醒，程序继续执行。

需要注意的是，Golang并非在每个指令执行周期都插入安全点，这会引入过大的开销。相反，它通过一种机制来推断哪些指令是安全点，即在可抢占点（preemption point）插入安全点。可抢占点是指一个goroutine可以被抢占的地方，例如函数的调用点、循环的迭代点、channel操作点等。当GC需要运行时，它会通过停止-复制的方式来中断正在执行的goroutine，然后扫描其中的对象，最后再将goroutine重新开始。

通过插入安全点，Golang能够在保证程序执行效率的同时，进行垃圾回收和其他系统操作，提高了性能和可伸缩性。

总之，safe point是Golang中的同步点，用于确保垃圾回收等操作可以安全地进行。通过在适当的位置插入安全点，Golang能够在保证程序执行效率的同时，进行必要的系统操作，提高了性能和可伸缩性。

## G的抢占 

一个G可以在任何 `安全点(safe-point)` 被抢占，目前安全点可以分为以下几类：

 1. 阻塞安全点出现在 goroutine 被取消调度、同步阻塞或系统调用期间；
 2. 同步安全点出现在运行goroutine检查抢占请求时;
 3. 异步安全点出现在用户代码中的任何指令上，其中G可以安全的暂停且可以保证堆栈和寄存器扫描找到 `stack root`（这个很重要,GC扫描开始的地方）。`runtime` 可以通过一个信号在一个异步安全点暂停一个G。

这里将安全点分为 `阻塞安全点`、`同步安全点` 和 `异步安全点`，每种安全点都出现在不同的场景。

`阻塞安全点` 和 `同步安全点`，一个G的CPU状态是最小的(_无法理解这里最小的意思_）。垃圾回收器拥有整个stack的完整信息。这样就有可能使用最小的空间重新调度G，并精确的扫描G的 栈。

`同步安全点` 是通过在重载函数序言中`stack bound chec`k(栈边界检查）实现的。在下一个同步安全点抢占G，runtime 在G的 stack绑定一个值，该值将导致下一个 `stack bound check` 失败，从而进入栈的增涨实现，此实现将检测到它实际上是抢占并重写向到抢占处理逻辑。

`异步安全点` 抢占是通过操作系统（如：信号）挂起一个线程并检查它的状态以确定G是否处于一个异步安全点。由于挂起线程本身是异步的，它将检查运行的G是否需要被抢占，这将引起一些改变。如果所有条件都满足，它将调整信号上下文，使其看起来像刚刚发起调用的`asyncPreempt`（异步抢占）信号线程并恢复此线程。`asyncPreempt` 溢出所有寄存器并进入调度程序。

（另一种方法是抢占信号处理程序本身。这将允许操作系统保存和恢复寄存器状态，运行时只需要知道如何从信号上下文中提取可能包含指针的寄存器。但是，这将为每个抢占的G消耗一个M，并且调度器本身并不是设计为从信号处理程序运行的，因为它倾向于在抢占路径中分配内存和启动线程）

## 暂停状态 

对 G 的暂停状态没有使用一个单独的变量来表示，而是通过一个 `suspendGState` 的结构体来表示。

```
type suspendGState struct {
	g *g
	dead bool
	stopped bool
}
```

字段意义：

 * `g` 表示当前暂停的G，将其放在状态结构体中，这样只需要一个结构体就可以了，不需要再单独占用一个参数来表示暂停的哪个G；
 * `dead` 表示当前G并没有暂停，而是处于 `_Gdead` 状态。这个 G 可以以后被复用，因为调用者不能一直认为它是 `_Gdead` 状态，见G的状态流转图；
 * `stopped` 表示通过 `g.preemptStop` 将G转换为 `_Gwaiting` 状态，因此负责在完成时做好准备

## 暂停G (suspendG) 

在安全点暂停G将返回一个 `suspendGState` 结构体的状态值，调用者在此期间将一直拥有此G的读权限，直到恢复 `resumeG` 为止。

多个调用者在同一时间试图suspend同一个G时，它是安全的。goroutine 可以在后续成功挂起操作之间执行。当前实现授予对G的独占访问权限，所以多个调用者将会序列化。但是，其目的是授予共享read权限，所以不要依赖独占访问。

suspend操作必须在系统栈执行，并且在M（如果有的话）上的用户goroutine必须处于一个可抢占的状态。这样可以防止两个goroutine试图互相挂起并且都处于非抢占状态时出现死锁。有其它的方式来解决这个死锁，但看起来非常的简单。

```
// go:systemstack
func suspendG(gp *g) suspendGState {
	// 当前暂停的G正是自己,且自己还处于_Grunning，直接抛出异常
	if mp := getg().m; mp.curg != nil && readgstatus(mp.curg) == _Grunning {
		throw("suspendG from non-preemptible goroutine")
	}

	const yieldDelay = 10 * 1000
	var nextYield int64

	stopped := false
	var asyncM *m
	var asyncGen uint32
	var nextPreemptM int64
	for i := 0; ; i++ {
		switch s := readgstatus(gp); s {
		default:
			if s&_Gscan != 0 {
				break
			}

			dumpgstatus(gp)
			throw("invalid g status")
		case _Gdead:
			return suspendGState{dead: true}
		case _Gcopystack:

		case _Gpreempted:
			// 抢占状态
			if !casGFromPreempted(gp, _Gpreempted, _Gwaiting) {
				break
			}

			stopped = true

			s = _Gwaiting
			fallthrough
		case _Grunnable, _Gsyscall, _Gwaiting:
			if !castogscanstatus(gp, s, s|_Gscan) {
				break
			}

			gp.preemptStop = false
			gp.preempt = false
			gp.stackguard0 = gp.stack.lo + _StackGuard
			return suspendGState{g: gp, stopped: stopped}
		case _Grunning:
			if gp.preemptStop && gp.preempt && gp.stackguard0 == stackPreempt && asyncM == gp.m && atomic.Load(&asyncM.preemptGen) == asyncGen {
				break
			}

			// Temporarily block state transitions.
			if !castogscanstatus(gp, _Grunning, _Gscanrunning) {
				break
			}

			// Request synchronous preemption.
			gp.preemptStop = true
			gp.preempt = true
			gp.stackguard0 = stackPreempt

			// Prepare for asynchronous preemption.
			asyncM2 := gp.m
			asyncGen2 := atomic.Load(&asyncM2.preemptGen)
			needAsync := asyncM != asyncM2 || asyncGen != asyncGen2
			asyncM = asyncM2
			asyncGen = asyncGen2

			casfrom_Gscanstatus(gp, _Gscanrunning, _Grunning)

			if preemptMSupported && debug.asyncpreemptoff == 0 && needAsync {
				now := nanotime()
				if now >= nextPreemptM {
					nextPreemptM = now + yieldDelay/2
					preemptM(asyncM)
				}
			}
		}

		if i == 0 {
			nextYield = nanotime() + yieldDelay
		}
		if nanotime() < nextYield {
			procyield(10)
		} else {
			osyield()
			nextYield = nanotime() + yieldDelay/2
		}
	}
```

整体流程是通过一个 for 方法，不断的检查G的状态并在合适的机会返回suspendGState。

 * `_Gdead` 已处于 dead状态，直接返回 `suspendGState{dead: true}`，注意这时没有g；
 * `_Gcopystack` 处于复制stack状态，当前处于栈的扩容或缩减，继续等待直到完成；
 * `_Gpreempted` 可抢占状态；将G变为 `_Gwaiting` 状态，同时设置变量 `stopped=true`。继续等待；
 * `_Grunnable`, `_Gsyscall`, `_Gwaiting` : 标记为扫描状态；取消抢占请求等，返回 `suspendGState{g: gp, stopped: true}`;
 * `_Grunning` 这里指非当前G的运行状态； 先将 `_Grunning` 变为 `_Gscanrunning`；设置`同步抢占`标记并做一些抢占准备，再恢复 `_Grunning` 状态;最后再发送异步抢占

这里提到过几个与转换G状态的函数，如` [casfrom_Gscanstatus()](https://github.com/golang/go/blob/go1.16.2/src/runtime/proc.go#L833-L861) `、` [castogscanstatus()](https://github.com/golang/go/blob/go1.16.2/src/runtime/proc.go#L863-L883) `、` [casGFromPreempted()](https://github.com/golang/go/blob/go1.16.2/src/runtime/proc.go#L956-L964) `、

## 恢复G (resumeG) 

所谓恢复G就是指对暂停状态的撤销，允许暂停的G从当前 `安全点(safe-point)` 继续执行。

```
func resumeG(state suspendGState) {
	if state.dead {
		// We didn't actually stop anything.
		return
	}

	gp := state.g
	switch s := readgstatus(gp); s {
	default:
		dumpgstatus(gp)
		throw("unexpected g status")

	case _Grunnable | _Gscan,
		_Gwaiting | _Gscan,
		_Gsyscall | _Gscan:
		casfrom_Gscanstatus(gp, s, s&^_Gscan)
	}

	if state.stopped {
		// We stopped it, so we need to re-schedule it.
		ready(gp, 0, true)
	}
}
```

主要是最后一句，调用 [`ready()`](https://github.com/golang/go/blob/go1.16.2/src/runtime/proc.go#L771-L792)，将其G设置为运行 `_Grunnable` 状态，这样 `G` 就可以在下次被立即执行。

## 总结 

可以看到对G的暂停和恢复，其实是对G 的状态进行改变。对于suspend操作只会在 `安全点` 才会发生，它会一直重试尝试着修改G的状态，同时会对一些抢占标记做一些修改直到修改成功为止。

## 参考资料 

 * [https://github.com/golang/go/blob/go1.16.2/src/runtime/preempt.go](https://github.com/golang/go/blob/go1.16.2/src/runtime/preempt.go)
 * [https://blog.haohtml.com/archives/21010](https://blog.haohtml.com/archives/21010)