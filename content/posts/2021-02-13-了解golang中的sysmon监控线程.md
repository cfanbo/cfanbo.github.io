---
title: 认识Golang中的sysmon监控线程
author: admin
type: post
date: 2021-02-13T10:02:02+00:00
url: /archives/22745
categories:
 - 程序开发
tags:
 - golang
 - runtime
 - sysmon

---
Go Runtime 在启动程序的时候，会创建一个独立的 `M` 作为监控线程，称为 `sysmon`，它是一个系统级的 `daemon` 线程。这个`sysmon` 独立于 GPM 之外，也就是说不需要P就可以运行，因此官方工具 `go tool trace` 是无法追踪分析到此线程（ [源码](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L4639-L4760)）。![](https://blogstatic.haohtml.com/uploads/2021/02/6ad0cfb3df2281110cf60630fcfb0e96.png)sysmon

在程序执行期间 `sysmon` 每隔 `20us~10ms` 轮询执行一次（ [源码](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L4652-L4659)），监控那些长时间运行的 G 任务, 然后设置其可以被强占的标识符，这样别的 `Goroutine` 就可以抢先进来执行。

```
// src/runtime/proc.go

// forcegcperiod is the maximum time in nanoseconds between garbage
// collections. If we go this long without a garbage collection, one
// is forced to run.
//
// This is a variable for testing purposes. It normally doesn't change.
var forcegcperiod int64 = 2 * 60 * 1e9

// Always runs without a P, so write barriers are not allowed.
//
//go:nowritebarrierrec
func sysmon() {
	......

	for {
		if idle == 0 { // start with 20us sleep...
			delay = 20
		} else if idle > 50 { // start doubling the sleep after 1ms...
			delay *= 2
		}
		if delay > 10*1000 { // up to 10ms
			delay = 10 * 1000
		}
		usleep(delay)

		......
	}

	......
}
```

说明一下，在 `sysmon()` 函数里有一个` [sched](https://github.com/golang/go/blob/go1.15.6/src/runtime/runtime2.go#L1041) ` 变量，这个是全局变量，它是整个调度器调度必须使用的一个全局变量，对应结构体为 ` [schedt](https://github.com/golang/go/blob/go1.15.6/src/runtime/runtime2.go#L698-L781) `, 对应字段注释我们 [golang中G、P、M 和 sched 三者的数据结构](https://blog.haohtml.com/archives/21010) 文章里已介绍过。

# sysmon工作职责 

在 GPM 模型中，我们知道当一个 `G` 由于网络请求或系统调用sysycall导致运行阻塞时，`go runtime scheduler` 会对其进行调度，把阻塞 `G` 切换出去，再拿一个新的`G`继续执行，始终让`P`处于全部运行状态。

那么切换出去的 `G` 什么时候再回来继续执行呢？这就是本文要讲的 `sysmon` 监控线程。![](https://blogstatic.haohtml.com/uploads/2021/04/d5142dae72a8b1bb39e0a991ddc68f09.png)sysmon 工作职责

## 网络轮询器监控 

在GPM模型中当由于 `网络请求` 而导致的运行阻塞时，调度器会让当前阻塞的`G`放入`网络轮询器（NetPoller）`中，由网络轮询器处理异步网络调用，从而出让 `P` 和 `M` ，这时 `M` 再从 `P` 的 LRQ 队列里取一个 `goroutine` 继续执行。当异步网络调用由网络轮询器完成后，再由`sysmon`监控线程将其切换回来继续执行。

从 `runtime` 的实现源码中可以看到, `sysmon` 监控线程会每隔 `20us~10ms` 轮询一次检查当前`网络轮询器`中的所有 `G` 距离上次 `runtime.netpoll` 被调用是否超过了`10ms`，如果当前G执行时间太长了，则通过 `injectglist()` 将其放入 `全局运行队列` ，等待下一次的继续执行。

> 对于 `sysmon` 线程来说，它的执行是完全脱离 `P` 运行的，所以不可能会放回本地运行队列。但在 `findrunnable() ` 函数里调用 `injectglist()` 函数是可以放入P

主要相关函数 [`netpoll()`](https://github.com/golang/go/blob/go1.15.6/src/runtime/netpoll_epoll.go#L101-L179) 和 [`injectglist()`](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L2542-L2605)，这里 `netpoll()` 函数是linux平台，不同平台所在的文件有所差异。

```
// src/runtime/proc.go

func sysmon() {
	......

	for {
		......

		// poll network if not polled for more than 10ms
		lastpoll := int64(atomic.Load64(&sched.lastpoll))
		if netpollinited() && lastpoll != 0 && lastpoll+10*1000*1000 < now {
			atomic.Cas64(&sched.lastpoll, uint64(lastpoll), uint64(now))
			list := netpoll(0) // non-blocking - returns list of goroutines
			if !list.empty() {
				// Need to decrement number of idle locked M's
				// (pretending that one more is running) before injectglist.
				// Otherwise it can lead to the following situation:
				// injectglist grabs all P's but before it starts M's to run the P's,
				// another M returns from syscall, finishes running its G,
				// observes that there is no work to do and no other running M's
				// and reports deadlock.
				incidlelocked(-1)
				injectglist(&list)
				incidlelocked(1)
			}
		}

		......
	}

	......
}
```

## 系统调用syscall监控 

如果是`系统调用`导致阻塞的话（P的状态为 `_Psyscall`），则从 `GMP` 模型中将 `GM` 切换出去，这样即保证了系统调用的继续执行，又节省出一个空闲的 `P`, 这种情况下 `P` 将重新找一个空闲 `M` （如果没有空间的 `M` 将重新创建一个 `M` ）,然后再优先从当前 `P` 的 `LRQ` 队列里取一个 `G` ，重新组成 `GMP` 模型继续执行。可以看到这里是充分将 `P` 资源利用起来。

```
// src/runtime/proc.go

func sysmon() {
	......

	for {
		......

		// retake P's blocked in syscalls
		// and preempt long running G's
		if retake(now) != 0 {
			idle = 0
		} else {
			idle++
		}

		......
	}

	......
}
```

`syscall` 主要通过调用 ` [retake()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L4769-L4840) ` 函数，抢占长时间处于 `_Psyscall` 状态的P。

```
// forcePreemptNS is the time slice given to a G before it is
// preempted.
const forcePreemptNS = 10 * 1000 * 1000 // 10ms

func retake(now int64) uint32 {
	n := 0

	// 全局p锁
	lock(&allpLock)

	// 遍历每个P
	for i := 0; i < len(allp); i++ {
		_p_ := allp[i]

		// 1. p == nil 说明通过runtime.GOMAXPROCS 动态对P进行了调大,但当前还未初始化使用 直接跳过
		if _p_ == nil {
			// This can happen if procresize has grown
			// allp but not yet created new Ps.
			continue
		}

		//
		pd := &_p_.sysmontick
		s := _p_.status
		sysretake := false

		// 2. P的状态为 _Prunning或_Psyscall的情况
		if s == _Prunning || s == _Psyscall {
			// Preempt G if it's running for too long.
			// 运行时间过长
			t := int64(_p_.schedtick)

			// 如果 p.sysmontick.schedtick != p.schedtick，则进行次数同步，并指定调度时间为now
			if int64(pd.schedtick) != t {
				pd.schedtick = uint32(t)
				pd.schedwhen = now
			} else if pd.schedwhen+forcePreemptNS <= now {
				// 如果 p上次调度时间 + 10ms <= now，说明下次调度的时候有点长，需要立即进行调度
				// 但是当前P 是_Psyscall 状态的话，并不会抢占P，因为这种情况下M并没有与P绑定
				preemptone(_p_)
				// In case of syscall, preemptone() doesn't
				// work, because there is no M wired to P.
				sysretake = true
			}
		}

		// 3. 系统调用
		if s == _Psyscall {
			// Retake P from syscall if it's there for more than 1 sysmon tick (at least 20us).
			t := int64(_p_.syscalltick)
			if !sysretake && int64(pd.syscalltick) != t {
				pd.syscalltick = uint32(t)
				pd.syscallwhen = now
				continue
			}
			// On the one hand we don't want to retake Ps if there is no other work to do,
			// but on the other hand we want to retake them eventually
			// because they can prevent the sysmon thread from deep sleep.
			if runqempty(_p_) && atomic.Load(&sched.nmspinning)+atomic.Load(&sched.npidle) > 0 && pd.syscallwhen+10*1000*1000 > now {
				continue
			}
			// Drop allpLock so we can take sched.lock.
			unlock(&allpLock)
			// Need to decrement number of idle locked M's
			// (pretending that one more is running) before the CAS.
			// Otherwise the M from which we retake can exit the syscall,
			// increment nmidle and report deadlock.
			incidlelocked(-1)
			if atomic.Cas(&_p_.status, s, _Pidle) {
				if trace.enabled {
					traceGoSysBlock(_p_)
					traceProcStop(_p_)
				}
				n++
				_p_.syscalltick++
				handoffp(_p_)
			}
			incidlelocked(1)
			lock(&allpLock)
		}

	}

	// 全局p锁
	unlock(&allpLock)
	return uint32(n)
}
```

队此之外还有由于原子、互斥量或通道操作调用导致  Goroutine  阻塞或直接调用 sleep 导致的阻塞也会导致调度器对G进行切换。

## 垃圾回收 

如果垃圾回收器超过`两分钟`没有执行的话，`sysmon` 监控线程也会强制进行GC，参考文章： [https://blog.haohtml.com/archives/23911](https://blog.haohtml.com/archives/23911)。

```
// src/runtime/proc.go

func sysmon() {
	......

	for {
		......

		// check if we need to force a GC
		if t := (gcTrigger{kind: gcTriggerTime, now: now}); t.test() && atomic.Load(&forcegc.idle) != 0 {
			lock(&forcegc.lock)
			forcegc.idle = 0
			var list gList
			list.push(forcegc.g)
			injectglist(&list)
			unlock(&forcegc.lock)
		}
		......
	}

	......
}
```

## 其它 

另外还有一些其它作用，如函数 [timeSleepUntil()](https://github.com/golang/go/blob/go1.16.2/src/runtime/proc.go#L5145) 用来获取 Timers 的下次执行时间

# 参考资料 

 * [https://zhuanlan.zhihu.com/p/248697371](https://zhuanlan.zhihu.com/p/248697371)
 * [https://zhuanlan.zhihu.com/p/27328476](https://zhuanlan.zhihu.com/p/27328476)
 * [Go netpoller 网络模型之源码全面解析](https://mp.weixin.qq.com/s/HNPeffn08QovQwtUH1qkbQ)
 * [Go 为什么这么“快”](https://mp.weixin.qq.com/s/ihJFa5Wir4ohhZUXVSBvMQ)