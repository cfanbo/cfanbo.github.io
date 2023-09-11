---
title: 'Runtime: Golang 定时器实现原理及源码解析'
author: admin
type: post
date: 2021-03-29T06:33:58+00:00
url: /archives/25962
categories:
 - 程序开发
tags:
 - golang
 - runtime
 - timer

---
定时器作为开发经常使用的一种数据类型，是每个开发者需要掌握的，对于一个高级开发很有必要了解它的实现原理，今天我们runtime源码来学习一下它的底层实现。

定时器分两种，分别为 Timer 和 Ticker，两者差不多，这里重点以Timer为例。

源文件位于 ` [src/time/sleep.go](https://github.com/golang/go/blob/go1.16.2/src/time/sleep.go) ` 和 ` [src/time/tick.go](https://github.com/golang/go/blob/go1.16.2/src/time/tick.go) ` 。 go version 1.16.2

# 数据结构 

`Timer` 数据结构

```
// src/runtime/sleep.go

// The Timer type represents a single event.
// When the Timer expires, the current time will be sent on C,
// unless the Timer was created by AfterFunc.
// A Timer must be created with NewTimer or AfterFunc.
type Timer struct {
	C <-chan Time
	r runtimeTimer
}
```

Timer 数据类型是表示单个事件。当计时器过期时，当前的时候将会发送到 Timer.C 通道，如果用 AfterFunc 创建计时器的话，则例外。

对于计时器必须由 ` [NewTimer()](https://github.com/golang/go/blob/go1.16.2/src/time/sleep.go#L84-L98) ` 或 ` [AfterFunc()](https://github.com/golang/go/blob/go1.16.2/src/time/sleep.go#L164-L181) ` 创建。

```
// Interface to timers implemented in package runtime.
// Must be in sync with ../runtime/time.go:/^type timer
type runtimeTimer struct {
	pp       uintptr
	when     int64
	period   int64
	f        func(interface{}, uintptr) // NOTE: must not be closure
	arg      interface{}
	seq      uintptr
	nextwhen int64
	status   uint32
}
```

字段说明

 * `PP` 当前对应的处理器P的指针
 * `when` 定时器被唤醒的时间
 * `period` 唤醒间隔时间，定时器为Timer数据类型时，此字段值为 0 时，否则为 `Ticker` 数据类型
 * `f` 唤醒后执行的函数，不能为 `闭包匿名函数`
 * `arg` 唤醒后执行的函数的参数
 * `seq` ?
 * `nextwhen` 当定时器状态为 `timerModifiedEarlier` 或 `timerModifiedLater` 时，需要使用此字段值刷新为 `when` 字段
 * `status` 定时器状态

[状态值](https://github.com/golang/go/blob/go1.16.2/src/runtime/time.go#L118-L160) 常量有以下几个

 * `timerNoStatus` = iota 初始化状态
 * `timerWaiting` 等待计时器启动,定时器在P堆中
 * `timerRunning` 定时器运行中，只很短暂的时间持有此状态
 * `timerDeleted` 定时器删除状态，后期不会运行，但仍会存在于P堆中
 * `timerRemoving` 正在移除，只有很短暂的时间持有此状态
 * `timerRemoved` 已移除，不在P堆中
 * `timerModifying` 正在修改中，只有很短暂的时间持有此状态
 * `timerModifiedEarlier` 定时器已修改为较早的时间，此时新的`when`值存储于 `nextwhen` 字段中。在P堆中，但有可以存储在错误的地方
 * `timerModifiedLater` 定时器修改为相同或较晚的状态，此时新的`when`值存储于 `nextwhen` 字段中。在P堆中，但有可以存储在错误的地方
 * `timerMoving` 定时器已修改并正在动

其中 当状态为 `timerModifiedEarlier` 或 `timerModifiedEarlier` 时，新的`when`值将使用 `nextwhen` 字段

这里定时器与P有关，下面我们看下P相关的字段

```
type p struct{
	...
	timer0When uint64
	timerModifiedEarliest uint64

	timersLock mutex
	timers []*timer
	numTimers uint32
	adjustTimers uint32
	deletedTimers uint32
	timerRaceCtx uintptr
	...
}
```

相关字段介绍，请参考 [https://blog.haohtml.com/archives/21010](https://blog.haohtml.com/archives/21010)

golang使用最小四叉堆数据结构(最小堆是指满足除了根节点以外的每个节点都不小于其父节点的堆)。golang []*****timer结构如下:![v2-640db017cb69978da3d397a84e405549_720w](https://blogstatic.haohtml.com/uploads/2021/03/d5acdf5e10819bc69eac386bb779f95e-22.jpg)golang存储定时任务结构

addtimer在堆中插入一个值，然后保持最小堆的特性，其实这个结构本质就是最小优先队列的一个应用，然后将时间转换一个绝对时间处理，通过睡眠和唤醒找出定时任务。

# 方法列表 

 * [type Timer][1]
 * [func AfterFunc(d Duration, f func()) *Timer][2]
 * [func NewTimer(d Duration) *Timer][3]
 * [func (t *Timer) Reset(d Duration) bool][4]
 * [func (t *Timer) Stop() bool][5]


`NewTimer` 创建一个计时咕器，在达到指定时间时，将发送当前时间到 Timer.C 通道
`AfterFunc` 创建一个计时器，但半不会发送值到 Timer.C 通道
`Reset` 重置修改定时时间参数 d，如果计时器已过期或已停止，则返回false，否则为true
`Stop` 停止计时器的运行。如果计时器已过期，则返回false，否则为 true。切记stop并不会关闭channel通道，否则有可能出现其它goroutine向一个已关闭的channel写数据导致的 panic。

# 实现原理 

## 创建和启动定时器 

上面我们介绍了创建一个定时器有两种方法，分别为 `NewTimer()` 和 `AfteFunc()`，两者只有一点不一样，就是是否向 `Timer.C` 通道里发送内容的区别，其它都是完全一样的，我们这里只介绍 `NewTimer()` 这个函数的实现。

```
func NewTimer(d Duration) *Timer {
	c := make(chan Time, 1)
	t := &Timer{
		C: c,
		r: runtimeTimer{
			when: when(d),
			f:    sendTime,
			arg:  c,
		},
	}
	startTimer(&t.r)
	return t
}
```

先声明接收数据的channel 变量`c`，然后创建一个 Timer数据类型指针，并将c赋值给 `timer.C` 字段，用以接收发送的数据。

调用 ` [runtime.startTimer()](https://github.com/golang/go/blob/go1.16.2/src/runtime/time.go#L206-L213) ` 函数启动定时器，它其实是对函数 ` [runtime.addtimer()](https://github.com/golang/go/blob/go1.16.2/src/runtime/time.go#L245-L273) ` 的封装。

在首次创建一个 `timer` 的时候，将当前 `timer` 添加到 `P` 的堆中。

```
// addtimer adds a timer to the current P.
// This should only be called with a newly created timer.
// That avoids the risk of changing the when field of a timer in some P's heap,
// which could cause the heap to become unsorted.
func addtimer(t *timer) {
	// when must be positive. A negative value will cause runtimer to
	// overflow during its delta calculation and never expire other runtime
	// timers. Zero will cause checkTimers to fail to notice the timer.
	if t.when <= 0 {
		throw("timer when must be positive")
	}
	if t.period < 0 {
		throw("timer period must be non-negative")
	}
	if t.status != timerNoStatus {
		throw("addtimer called with initialized timer")
	}
	t.status = timerWaiting

	when := t.when

	// 获取当前G所在的P
	pp := getg().m.p.ptr()

	// 加timer锁
	lock(&pp.timersLock)
	cleantimers(pp)
	doaddtimer(pp, t)
	unlock(&pp.timersLock)

	wakeNetPoller(when)
}
```

操作顺序为

 * 检查定时器状态和当前P的状态(初始化状态)是否满足条件
 * 获取当前G所在的P
 * 加P加 `timerLock` 锁
 * 调用 ` [cleantimers()](https://github.com/golang/go/blob/go1.16.2/src/runtime/time.go#L547-L600) ` 函数清除P队列头中的 `timers`，并将 `timer` 添加的P的最小堆中
 * 解 `timerLock` 锁
 * 调用 [`wakeNetPoller`](https://github.com/golang/go/blob/go1.16.2/src/runtime/proc.go#L2941-L2961) 函数，唤醒网络轮询器中休眠的线程,检查计时器被唤醒的时间（when）是否在当前轮询预期运行的时间（`pollerPollUntil`）内，若是则唤醒。

下面我们重点看两个函数

通过调用 `cleantimers()` 函数可以加速创建和删除计时器；将其放在堆里可以减慢`addtimer`。

```
func cleantimers(pp *p) {
	gp := getg()
	for {
		// 定时器数组为空，直接返回
		if len(pp.timers) == 0 {
			return
		}

		// This loop can theoretically run for a while, and because
		// it is holding timersLock it cannot be preempted.
		// If someone is trying to preempt us, just return.
		// We can clean the timers later.
		// 当前G被抢占了，直接返回
		if gp.preemptStop {
			return
		}

		// 获取列表数组的第一个定时器
		t := pp.timers[0]
		if t.pp.ptr() != pp {
			throw("cleantimers: bad p")
		}
		switch s := atomic.Load(&t.status); s {
		case timerDeleted:
			// timerDeleted => timerRemoving
			if !atomic.Cas(&t.status, s, timerRemoving) {
				continue
			}

			// 删除当前P的第一个定时器
			dodeltimer0(pp)

			// timerRemoving => timerRemoved
			if !atomic.Cas(&t.status, timerRemoving, timerRemoved) {
				badTimer()
			}
			atomic.Xadd(&pp.deletedTimers, -1)

		case timerModifiedEarlier, timerModifiedLater:
			if !atomic.Cas(&t.status, s, timerMoving) {
				continue
			}

			// Now we can change the when field.
			// 从t.nextwhen 字段中读取新的值到 t.when 字段
			t.when = t.nextwhen

			// Move t to the right position.
			// 先从P删除，再添加到P中
			dodeltimer0(pp)
			doaddtimer(pp, t)
			if s == timerModifiedEarlier {
				atomic.Xadd(&pp.adjustTimers, -1)
			}

			// timeMoving => timerWaiting
			if !atomic.Cas(&t.status, timerMoving, timerWaiting) {
				badTimer()
			}
		default:
			// Head of timers does not need adjustment.
			return
		}
	}
}
```

这里函数通过一个for循环，递归的删除P中头部第一个timer。

如果timer的状态为 `timerDeleted` 则先将其改为 `timerRemoving`， 再修改为 `timerRemoved`，并调用函数 ` [dodeltimer0()](https://github.com/golang/go/blob/go1.16.2/src/runtime/time.go#L391-L412) ` 将其从P堆中删除。

如果timer的状态为 `timerModifiedEarlier` 或 `timerModifiedLater`，则对 `when` 字段值进行修正(`t.when = t.nextwhen`），先从P中删除，再调用函数 ` [doaddtimer()](https://github.com/golang/go/blob/go1.16.2/src/runtime/time.go#L275-L295) ` 重新添加当前的P中，这里利用了小堆的特性(见函数 ` [siftupTimer()](https://github.com/golang/go/blob/go1.16.2/src/runtime/time.go#L1075-L1103) `)。

```
// doaddtimer adds t to the current P's heap.
// The caller must have locked the timers for pp.
func doaddtimer(pp *p, t *timer) {
	// Timers rely on the network poller, so make sure the poller
	// has started.
	if netpollInited == 0 {
		netpollGenericInit()
	}

	if t.pp != 0 {
		throw("doaddtimer: P already set in timer")
	}

	// 将定时器绑定P，并添加到P堆中
	t.pp.set(pp)
	i := len(pp.timers)
	pp.timers = append(pp.timers, t)

	// 维护 timer 在 最小堆中的位置
	siftupTimer(pp.timers, i)

	// 如果当前timer正好是堆第一个对象，则需要刷新 p.timer0When 字段值为 t.when
	if t == pp.timers[0] {
		atomic.Store64(&pp.timer0When, uint64(t.when))
	}
	atomic.Xadd(&pp.numTimers, 1)
}
```

## 停止定时器 

`timer.Stop(`) 函数是对 `time.stopTimer` 函数的封装。而在内部是调用了`deltimer()` 来实现的。

```
// src/runtime/time.go
// stopTimer stops a timer.
// It reports whether t was stopped before being run.
//go:linkname stopTimer time.stopTimer
func stopTimer(t *timer) bool {
	return deltimer(t)
}

// deltimer deletes the timer t. It may be on some other P, so we can't
// actually remove it from the timers heap. We can only mark it as deleted.
// It will be removed in due course by the P whose heap it is on.
// Reports whether the timer was removed before it was run.
func deltimer(t *timer) bool {
	for {
		switch s := atomic.Load(&t.status); s {
		case timerWaiting, timerModifiedLater:
			// Prevent preemption while the timer is in timerModifying.
			// This could lead to a self-deadlock. See #38070.
			mp := acquirem()
			if atomic.Cas(&t.status, s, timerModifying) {
				// Must fetch t.pp before changing status,
				// as cleantimers in another goroutine
				// can clear t.pp of a timerDeleted timer.
				tpp := t.pp.ptr()
				if !atomic.Cas(&t.status, timerModifying, timerDeleted) {
					badTimer()
				}
				releasem(mp)
				atomic.Xadd(&tpp.deletedTimers, 1)
				// Timer was not yet run.
				return true
			} else {
				releasem(mp)
			}
		case timerModifiedEarlier:
			// Prevent preemption while the timer is in timerModifying.
			// This could lead to a self-deadlock. See #38070.
			mp := acquirem()
			if atomic.Cas(&t.status, s, timerModifying) {
				// Must fetch t.pp before setting status
				// to timerDeleted.
				tpp := t.pp.ptr()
				atomic.Xadd(&tpp.adjustTimers, -1)
				if !atomic.Cas(&t.status, timerModifying, timerDeleted) {
					badTimer()
				}
				releasem(mp)
				atomic.Xadd(&tpp.deletedTimers, 1)
				// Timer was not yet run.
				return true
			} else {
				releasem(mp)
			}
		case timerDeleted, timerRemoving, timerRemoved:
			// Timer was already run.
			return false
		case timerRunning, timerMoving:
			// The timer is being run or moved, by a different P.
			// Wait for it to complete.
			osyield()
		case timerNoStatus:
			// Removing timer that was never added or
			// has already been run. Also see issue 21874.
			return false
		case timerModifying:
			// Simultaneous calls to deltimer and modtimer.
			// Wait for the other call to complete.
			osyield()
		default:
			badTimer()
		}
	}
}
```

对于timer 的删除不能直接从堆中删除，因为它可能不在当前的P，而是在在其它的P堆上，所以只能将其标记为删除状态，在适当的时候将自动删除。

根据当前定时器不同状态进行以下处理：

 * timerWaiting, timerModifiedLater => timerModifying => timerDeleted
 * timerModifiedEarlier => timerModifying => timerDeleted
 * timerDeleted, timerRemoving, timerRemoved 直接返回false
 * timerRunning, timerMoving: 正在运动或正在移动，调用 `osyield()` 下次调度
 * timerNoStatus 初始化状态，返回false
 * timerModifying 表示正在调用 `deltimer` 或 `modtimer` ,等待完成
 * 其它 异常

# 重置和修改 

对于 `timer.Reset()` 函数，对应的是`resetTimer`,而其内部实现为

```
func resettimer(t *timer, when int64) bool {
	return modtimer(t, when, t.period, t.f, t.arg, t.seq)
}
```

这与修改操作对应的是同一个 ` [modtimer()](https://github.com/golang/go/blob/go1.16.2/src/runtime/time.go#L414-L536) ` 函数。

对于这个函数的一般会被网络轮询、`timer.Ticker.Reset` 或 `timer.Timer.Reset` 函数调用。

先是用一个for自旋修改定时器状态

```
func modtimer(t *timer, when, period int64, f func(interface{}, uintptr), arg interface{}, seq uintptr) bool {
	for {
		switch status = atomic.Load(&t.status); status {
		case timerWaiting, timerModifiedEarlier, timerModifiedLater:
		case timerNoStatus, timerRemoved:
		case timerDeleted:
		case timerRunning, timerRemoving, timerMoving:
		case timerModifying:
		default:
	}
}
```

然后

```
func modtimer(t *timer, when, period int64, f func(interface{}, uintptr), arg interface{}, seq uintptr) bool {
	......

	t.period = period
	t.f = f
	t.arg = arg
	t.seq = seq

// 如果已从P中移除，重新加入到P中 timerModifiying => timerWaiting
	if wasRemoved {
		t.when = when
		pp := getg().m.p.ptr()
		lock(&pp.timersLock)
		doaddtimer(pp, t)
		unlock(&pp.timersLock)
		if !atomic.Cas(&t.status, timerModifying, timerWaiting) {
			badTimer()
		}
		releasem(mp)
		wakeNetPoller(when)
	} else {
		// The timer is in some other P's heap, so we can't change
		// the when field. If we did, the other P's heap would
		// be out of order. So we put the new when value in the
		// nextwhen field, and let the other P set the when field
		// when it is prepared to resort the heap.
		t.nextwhen = when

// 如果修改后的时间<修改之前的时间，则修改状态为 timerModifiedEarlier
		newStatus := uint32(timerModifiedLater)
		if when < t.when {
			newStatus = timerModifiedEarlier
		}

		tpp := t.pp.ptr()

		// Update the adjustTimers field.  Subtract one if we
		// are removing a timerModifiedEarlier, add one if we
		// are adding a timerModifiedEarlier.
		adjust := int32(0)
		if status == timerModifiedEarlier {
			adjust--
		}
		if newStatus == timerModifiedEarlier {
			adjust++
			updateTimerModifiedEarliest(tpp, when)
		}
		if adjust != 0 {
			atomic.Xadd(&tpp.adjustTimers, adjust)
		}

		// Set the new status of the timer.
		if !atomic.Cas(&t.status, timerModifying, newStatus) {
			badTimer()
		}
		releasem(mp)

		// If the new status is earlier, wake up the poller.
		// 如果新状态提前 timerModifiedEarlier，则调用 wakeNetPooler 唤醒网络轮询器中休眠的线程,检查计时器被唤醒的时间（when）是否在当前轮询预期运行的时间（pollerPollUntil）内，若是则唤醒。
		if newStatus == timerModifiedEarlier {
			wakeNetPoller(when)
		}
	}

	return pending
}
```

如果当前定时器已从P堆中删除，则重新加入P堆中；
如果修改后的时间提前了，则修改状态为 timerModifiedEarlier，同时唤醒netpool中休眠的线程。

# 定时器的触发机制 

共分两种方式，分别为 `调度器触发` 和 `监控线程sysmon` 触发，两者主要是通过调用函数 checkTimers() 来实现的。

## 调度器触发 

主要有两个地方会检查计时器，一个是 ` [runtime.schedule](https://github.com/golang/go/blob/go1.16.2/src/runtime/proc.go#L3049-L3165) `，另一个是 ` [findrunnable](https://github.com/golang/go/blob/go1.16.2/src/runtime/proc.go#L2552-L2918) `。

[`runtime.schedule()`][6] :

```
func schedule() {
	......
top:
	pp := _g_.m.p.ptr()
	pp.preempt = false

	if sched.gcwaiting != 0 {
		gcstopm()
		goto top
	}
	if pp.runSafePointFn != 0 {
		runSafePointFn()
	}

	// Sanity check: if we are spinning, the run queue should be empty.
	// Check this before calling checkTimers, as that might call
	// goready to put a ready goroutine on the local run queue.
	if _g_.m.spinning && (pp.runnext != 0 || pp.runqhead != pp.runqtail) {
		throw("schedule: spinning with local work")
	}

	// 定时器执行
	checkTimers(pp, 0)

	var gp *g
	var inheritTime
	......
}
```

` [findrunnable](https://github.com/golang/go/blob/go1.16.2/src/runtime/proc.go#L2552-L2918) `

```
func findrunnable() (gp *g, inheritTime bool) {
	_g_ := getg()
top:
	_p_ := _g_.m.p.ptr()
	if sched.gcwaiting != 0 {
		gcstopm()
		goto top
	}
	if _p_.runSafePointFn != 0 {
		runSafePointFn()
	}

	// 检查当前P下面的计时器
	now, pollUntil, _ := checkTimers(_p_, 0)

	...
	// 检查要窃取P下面的计时器
	for i := 0; i < stealTries; i++ {
		stealTimersOrRunNextG := i == stealTries-1

		for enum := stealOrder.start(fastrand()); !enum.done(); enum.next() {
		...
		if stealTimersOrRunNextG && timerpMask.read(enum.position()) {
				tnow, w, ran := checkTimers(p2, now)
				now = tnow
				if w != 0 && (pollUntil == 0 || w < pollUntil) {
					pollUntil = w
				}
			}

}
```

对于 findrunnable() 函数来讲，主要是先从当前P中获取时会检查一次；另外如果当前P没有G的话，会进行窃取P，此时也会检查窃取P下面的计时器。

` [checkTimers()](https://github.com/golang/go/blob/go1.16.2/src/runtime/proc.go#L3181-L3244) ` 函数实现

```
func checkTimers(pp *p, now int64) (rnow, pollUntil int64, ran bool) {
	// If it's not yet time for the first timer, or the first adjusted
	// timer, then there is nothing to do.
	next := int64(atomic.Load64(&pp.timer0When))
	nextAdj := int64(atomic.Load64(&pp.timerModifiedEarliest))
	if next == 0 || (nextAdj != 0 && nextAdj < next) {
		next = nextAdj
	}

	// 等于0 说明没有定时器
	if next == 0 {
		// No timers to run or adjust.
		return now, 0, false
	}

	if now == 0 {
		now = nanotime()
	}
	// 时间未到
	if now < next {
		// Next timer is not ready to run, but keep going
		// if we would clear deleted timers.
		// This corresponds to the condition below where
		// we decide whether to call clearDeletedTimers.
		// 如果需要删除的定时器个数 <= 总定时器数量的1/4，则直接返回
		if pp != getg().m.p.ptr() || int(atomic.Load(&pp.deletedTimers)) <= int(atomic.Load(&pp.numTimers)/4) {
			return now, next, false
		}
	}

	lock(&pp.timersLock)

	if len(pp.timers) > 0 {
		adjusttimers(pp, now)
		for len(pp.timers) > 0 {
			// Note that runtimer may temporarily unlock
			// pp.timersLock.
			// 查找P堆中需要执行的定时器
			if tw := runtimer(pp, now); tw != 0 {
				if tw > 0 {
					pollUntil = tw
				}
				break
			}
			ran = true
		}
	}

	// If this is the local P, and there are a lot of deleted timers,
	// clear them out. We only do this for the local P to reduce
	// lock contention on timersLock.
	// 是当前正在使用中的P，如果要删除的timers > 总数量的1/4，则进行清理删除
	if pp == getg().m.p.ptr() && int(atomic.Load(&pp.deletedTimers)) > len(pp.timers)/4 {
		clearDeletedTimers(pp)
	}

	unlock(&pp.timersLock)

	return now, pollUntil, ran
}
```

对于checkTimers 函数的工作主要有

 * 获取P下面下次运行计时器的时间，如果P下没有计时器，则直接返回
 * 如果需要删除的定时器个数 <= 总计时器数量的1/4，则直接返回
 * 如果有计时器，则调用函数 ` [adjusttimers()](https://github.com/golang/go/blob/go1.16.2/src/runtime/time.go#L656-L747) ` 进行调用（堆维护)
 * 如果要检查的P正好是当前正在使用的P，这种情况下如果要删除的 timers > 总数量的1/4，则调用函数 ` [clearDeletedtimers](https://github.com/golang/go/blob/go1.16.2/src/runtime/time.go#L904-L992)()` 函数进行清理删除

## 监控线程触发 

```
func sysmon() {
	...
	//  It does not reset idle when waking
	// from a timer to avoid adding system load to applications that spend
	// most of their time sleeping.
	for {
		now := nanotime()
		if debug.schedtrace <= 0 && (sched.gcwaiting != 0 || atomic.Load(&sched.npidle) == uint32(gomaxprocs)) {
			lock(&sched.lock)
			if atomic.Load(&sched.gcwaiting) != 0 || atomic.Load(&sched.npidle) == uint32(gomaxprocs) {
				syscallWake := false

				// 获取下一次计时器启动的时间和持有该堆的P
				next, _ := timeSleepUntil()
				if next > now {
					atomic.Store(&sched.sysmonwait, 1)
					unlock(&sched.lock)
					// Make wake-up period small enough
					// for the sampling to be correct.

					// 计算离下次启动计时器的时间
					sleep := forcegcperiod / 2
					if next-now < sleep {
						sleep = next - now
					}
					shouldRelax := sleep >= osRelaxMinNS
					if shouldRelax {
						osRelax(true)
					}

					// 休眠一段时间，唤醒后将直接执行堆中的timers
					syscallWake = notetsleep(&sched.sysmonnote, sleep)
					mDoFixup()
					if shouldRelax {
						osRelax(false)
					}
					lock(&sched.lock)
					atomic.Store(&sched.sysmonwait, 0)
					noteclear(&sched.sysmonnote)
				}
				if syscallWake {
					idle = 0
					delay = 20
				}
			}
			unlock(&sched.lock)
		}
	}
	...
}
```

 1. 调用 ` [timeSleepUntil()](https://github.com/golang/go/blob/go1.16.2/src/runtime/time.go#L1042-L1073) ` 函数，遍历所有的P，找出下次最先执行(时间值最小)的时间和其所在的P
 2. 调用 ` [notesleep()](https://github.com/golang/go/blob/go1.16.2/src/runtime/lock_sema.go#L277-L284) ` 函数休眠一段时间。待唤醒后将自动执行堆上的timers。另外还有一个 ` [notesleepg()](https://github.com/golang/go/blob/go1.16.2/src/runtime/lock_sema.go#L286-L298) `函数，区域是 `notesleep()` 调用者是否为g0

说明一下，在sysmon 监控线程里没有找到过期 timer 情况下的处理逻辑，只有未过期的处理逻辑。

 [1]: https://pkg.go.dev/time#Timer
 [2]: https://pkg.go.dev/time#AfterFunc
 [3]: https://pkg.go.dev/time#NewTimer
 [4]: https://pkg.go.dev/time#Timer.Reset
 [5]: https://pkg.go.dev/time#Timer.Stop
 [6]: https://github.com/golang/go/blob/go1.16.2/src/runtime/proc.go#L3049-L3165