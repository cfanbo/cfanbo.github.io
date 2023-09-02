---
title: 'Runtime: Golang同步原语Mutex源码分析'
author: admin
type: post
date: 2021-03-19T03:15:16+00:00
url: /archives/24013
categories:
 - 程序开发
tags:
 - golang
 - mutex

---
在 `sync` 包里提供了最基本的同步原语，如互斥锁 `Mutex`。除 `Once` 和 `WaitGroup` 类型外，大部分是由低级库提供的，更高级别的同步最好是通过 `channel` 通讯来实现。

`Mutex` 类型的变量默认值是未加锁状态，在第一次使用后，此值将`不得`复制，这点切记！！！

本文基于go version: 1.16.2

Mutex 锁实现了 `Locker` 接口。

```
// A Locker represents an object that can be locked and unlocked.
type Locker interface {
	Lock()
	Unlock()
}
```

## 锁的模式 

为了互斥公平性，Mutex 分为 `正常模式` 和 `饥饿模式` 两种。

### 正常模式 

在正常模式下，等待者 `waiter` 会进入到一个`FIFO`队列，在获取锁时`waiter`会按照先进先出的顺序获取。当唤醒一个`waiter` 时它被并不会立即获取锁，而是要与`新来的goroutine`竞争，这种情况下新来的goroutine比较有优势，主要是因为它已经运行在CPU，可能它的数量还不少，所以`waiter`大概率下获取不到锁。在这种`waiter`获取不到锁的情况下，`waiter`会被添加到队列的前面。如果`waiter`获取不到锁的时间超出了1毫秒，它将被切换为饥饿模式。

这里的 `waiter` 是指新来一个goroutine 时会尝试一次获取锁，如果获取不到我们就视其为`watier`，并将其添加到FIFO队列里。

### 饥饿模式 

在正常模式下，每次新来的goroutine都会抢走锁，就这会导致一些 `waiter` 永远也获取不到锁，产生饥饿问题。所以为了应对高并发抢锁场景下的公平性，官方引入了饥饿模式。

在饥饿模式下，锁将直接交给队列最前面的`waiter`。新来的goroutine即使在锁未被持有情况下也不会参与竞争锁，同时也不会进行自旋，而直接将其添加到队列的尾部。

如果`拥有锁的waiter`发现有以下两种情况，它将切换回正常模式：

 1. 它是队列里的最后一个waiter，再也没有其它waiter
 2. 等待时间小于1毫秒

### 模式区别 

`正常模式` 拥有更好的性能，因为即使等待队列里有抢锁的 `waiter`，由于新来的`goroutine` 正在CPU中运行，所以优先获取到锁。
`饥饿模式` 是对公平性和性能的一种平衡，它避免了某些 `goroutine` 长时间的等待锁。在饥饿模式下，优先处理的是那些一直在等待的 `waiter`。饥饿模式在一定机时会切换回正常模式。

## 数据结构 

Mutex 锁的方法

```
type Mutex
    func (m *Mutex) Lock()
    func (m *Mutex) Unlock()
```

主要有 [`Lock()`](https://github.com/golang/go/blob/go1.16.2/src/sync/mutex.go#L69-L82) 和 ` [Unlock()](https://github.com/golang/go/blob/go1.16.2/src/sync/mutex.go#L173-L192) ` 两个方法，实现了 ` [Locker](https://github.com/golang/go/blob/go1.16.2/src/sync/mutex.go#L30-L34) ` 接口。

Mutex 结构体

```
// A Mutex is a mutual exclusion lock.
// The zero value for a Mutex is an unlocked mutex.
//
// A Mutex must not be copied after first use.
type Mutex struct {
	state int32 // 4字节
	sema  uint32 // 4字节
}
```

主要有 `state` 和 `sema` 两个字段组成，共8字节。其中 `sema` 是用于控制锁状态的信号量，`state` 表示锁的状态。

这里的 `state` 字段是一个复合型的字段，即一个字段包含多个意义，这样就可使用最小的内存来表示更多的意义，实现互斥锁。表现形式为 `state` 字段值的 `二进制` 格式。目前共分四部分，其中低三位分别表示`mutexed`、`mutexWoken` 和 `mutexStarving`，剩下的位则用来表示当前共有多少个goroutine 在等待锁。![e0c23794c8a1d355a7a183400c036276](https://blogstatic.haohtml.com/uploads/2021/03/d3abdc9d4561946a80d42fb9e06e5d22-58.jpg)mutex state

在默认情况下，互斥锁的所有状态位都是0， 不同的位表示了不同的状态

 * `mutexLocked` 表示锁定状态
 * `mutexWoken` 表示waiter 唤醒状态
 * `mutexStarving` 表示饥饿状态
 * `mutexWaiters`表示waiter的个数，最大允许记录 1<<(32-3) -1个goroutine

## 实现原理 

在此之前先了解几个与Mutex锁相关的 [常量](https://github.com/golang/go/blob/go1.16.2/src/sync/mutex.go#L36-L67)

```
const (
	mutexLocked = 1 << iota // mutex is locked
	mutexWoken // 2 二进制 0010
	mutexStarving // 4 二进制 0100
	mutexWaiterShift = iota // 3
	starvationThresholdNs = 1e6 // 1毫秒，用来与waiter的等待时间做比较
)
```

其中前四个常量会参与位运算。

对于加锁取与解锁主要有两个步骤，分别为 `fast path` 和 `slow path` 两个方法。我们看一下加锁 。

### 加锁 

加锁方法对应的是 ` [Lock()](https://github.com/golang/go/blob/go1.16.2/src/sync/mutex.go#L69-L82) ` ，其中还有一个私有方法 ` [lockSlow()](https://github.com/golang/go/blob/go1.16.2/src/sync/mutex.go#L84-L171) `。

```
// Lock locks m.
// If the lock is already in use, the calling goroutine
// blocks until the mutex is available.
func (m *Mutex) Lock() {
	// Fast path: grab unlocked mutex.
	// 锁未被持有，则直接获取持有权
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
		if race.Enabled {
			race.Acquire(unsafe.Pointer(m))
		}
		return
	}
	// Slow path (outlined so that the fast path can be inlined)
	// 尝试自旋竞争或饥饿状态下饥饿goroutine竞争
	m.lockSlow()
}

```

先是 `fast path` 幸运路径，使用 CAS 直接获取锁，如果运气好的话会直接获取到锁并返回。如果获取不到则 `slow path`，这里是 [`lockSlow()`](https://github.com/golang/go/blob/go1.16.2/src/sync/mutex.go#L84-L171) 函数，其实现比较复杂。

```
func (m *Mutex) lockSlow() {
	var waitStartTime int64 // 当前waiter开始等待时间
	starving := false // 当前饥饿状态
	awoke := false // 当前唤醒状态
	iter := 0 // 当前自旋次数
	old := m.state // 当前锁的状态
	for {
		// Don't spin in starvation mode, ownership is handed off to waiters
		// so we won't be able to acquire the mutex anyway.
		// 在饥饿模式下不需要自旋，直接将锁移交给waiter（队列头部的waiter)，因此新来的goroutine永远也不会获取锁

		// 正常模式下，锁被其它goroutine持有,如果当前允许spinning, 则尝试进行自旋
		if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {
			// Active spinning makes sense.
			// Try to set mutexWoken flag to inform Unlock
			// to not wake other blocked goroutines.
			if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
				atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
				awoke = true // 设置当前goroutine唤醒成功
			}
			runtime_doSpin() // 自旋
			iter++ // 当前自旋次数+1
			old = m.state // 当前goroutine再次获取锁的状态，之后会检查是否锁被释放了
			continue // 重新判断spinning
		}

		......
	}

	......
}
```

如果当前锁被其它goroutine持有(`低一位为1`)且处于正常模式(`低三位为0`)，且当前还允许自旋则进行自旋操作。

重点介绍这下这块的位运算逻辑，`mutexLocked` 和 `mutexStarving` 这两个位分别代表了锁 `是否被持有` 和 `饥饿状态` ，它们的二进制值表示 `0001`和 `0100`。

判断条件 `old&(mutexLocked|mutexStarving) == 0001` 可以转化为

```
old & (0001 | 0100)
old & 0101
```

如果 `old & 0101 = 0001` ，由此计算得知 `old` 的值必须是低一位为1，低三位为0，即处于加锁状态和饥饿状态。

根据当前 `state` 状态分为两种情况：

**一、 加锁状态且允许 spinning**

**1）**先通过 `CAS` 设置 `mutexWoken` 标记以通知解锁（位运算），并将当前 `goroutine` 的唤醒变量 `awoke` 设置为 `true`，以保证 g 只执行次 CAS 。

我们看下这块的位运算逻辑，判断条件共有四个

 * `!awoke` 表示waiter 处于未唤醒状态
 * `old&mutexWoken == 0` 表示未唤醒状态。old & 0010 = 0 ，则表示低第二位值为0表示未唤醒状态
 * `old>>mutexWaiterShift != 0` 即 `m.state >> 3` 的值不等于0，则说明当前 `waitersCount > 0`, 表示当前存在等待释放锁的 goroutine
 * `atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken)` 设置唤醒位的值为1。如0001 | 0010 = 0011

**2）**然后调用函数 `runtime_doSpin()` 执行自旋；

**3）**并更新自旋次数，同步新的状态 m.state。此函数内部会执行30次的 `PAUSE` 指令。

**4）**重复上面的三个步骤

对于这种情况最多执行四次（原因下面会介绍），以后就是执行下面逻辑

**二、非加锁状态或不允许 spinning**

```
func (m *Mutex) lockSlow() {
	for {
		......

		// 当前goroutine的 m.state 新状态
		new := old
		// Don't try to acquire starving mutex, new arriving goroutines must queue.
		// 如果当前处于正常模式，则加锁
		if old&mutexStarving == 0 {
			new |= mutexLocked
		}

		// 当前处于饥饿模式，则更新waiters数量 +1
		if old&(mutexLocked|mutexStarving) != 0 {
			new += 1 << mutexWaiterShift
		}

		// The current goroutine switches mutex to starvation mode.
		// But if the mutex is currently unlocked, don't do the switch.
		// Unlock expects that starving mutex has waiters, which will not
		// be true in this case.
		// 当前goroutine处于饥饿状态且锁被其它goroutine持有，新状态则更新锁为饥饿模式
		if starving && old&mutexLocked != 0 {
			new |= mutexStarving
		}
		// 当前goroutine的waiter被唤醒,则重置flag
		if awoke {
			// The goroutine has been woken from sleep,
			// so we need to reset the flag in either case.
			// 唤醒状态不一致，直接抛出异常
			if new&mutexWoken == 0 {
				throw("sync: inconsistent mutex state")
			}
			// 新状态清除唤醒标记
			new &^= mutexWoken
		}

		// CAS更新 m.state 状态成功
		if atomic.CompareAndSwapInt32(&m.state, old, new) {
			// 锁已被释放且为正常模式
			if old&(mutexLocked|mutexStarving) == 0 {
				// 通过 CAS 函数获取了锁，直接中止返回
				break // locked the mutex with CAS
			}

			// If we were already waiting before, queue at the front of the queue.
			// waitStartTime != 0 说明当前已处于等待状态
			queueLifo := waitStartTime != 0
			// 首次设置当前goroutine的开始等待时间
			if waitStartTime == 0 {
				waitStartTime = runtime_nanotime()
			}

			// queueLifo为true，说明已经等待了一会，本次循环则直接将waiter添加到等待队列的头部，使用信号量
			runtime_SemacquireMutex(&m.sema, queueLifo, 1)

			// 如果当前goroutine的等待时间>1毫秒则视为饥饿状态
			starving = starving || runtime_nanotime()-waitStartTime > starvationThresholdNs
			old = m.state

			// 如果处于饥饿状态(有可能等待时间>1毫秒)
			if old&mutexStarving != 0 {
				if old&(mutexLocked|mutexWoken) != 0 || old>>mutexWaiterShift == 0 {
					throw("sync: inconsistent mutex state")
				}

				// 加锁并且将waiter数减1(暂时未理解这块）
				delta := int32(mutexLocked - 1<<mutexWaiterShift)

				// 当前goroutine非饥饿状态 或者 等待队列只剩下一个waiter，则退出饥饿模式(清除饥饿标识位)
				if !starving || old>>mutexWaiterShift == 1 {
					delta -= mutexStarving
				}
				// 更新状态值并中止for循环
				atomic.AddInt32(&m.state, delta)
				break
			}

			// 设置当前goroutine为唤醒状态，且重置自璇次数
			awoke = true
			iter = 0
		} else {
			old = m.state
		}
	}

}
```

可以看到主要实现原来就是通过一个for循环实现的，正常模式下可能发生spinning，而允许自旋必须有四个条件，最多允许有四次spinning机会，否则将转为饥饿模式。饥饿模式下，需要对waiter数据进行累加。而当队列里只剩下一个它自己一个waiter的时候，会恢复为正常模式。每次是计算出了新的状态值new，下面通过 cas 实现更新状态，如果更新失败，则读取新的锁状态m.state并开始新一轮的for循环逻辑。这里的逻辑比较复杂不是太容易理解。

函数 ` [runtime_canSpin()](https://github.com/golang/go/blob/go1.16.2/src/runtime/proc.go#L5577-L5593) ` 判断是否满足自旋条件。

```
// src/runtime/proc.go

// Active spinning for sync.Mutex.
//go:linkname sync_runtime_canSpin sync.runtime_canSpin
//go:nosplit
func sync_runtime_canSpin(i int) bool {
	// sync.Mutex is cooperative, so we are conservative with spinning.
	// Spin only few times and only if running on a multicore machine and
	// GOMAXPROCS>1 and there is at least one other running P and local runq is empty.
	// As opposed to runtime mutex we don't do passive spinning here,
	// because there can be work on global runq or on other Ps.
	if i >= active_spin || ncpu <= 1 || gomaxprocs <= int32(sched.npidle+sched.nmspinning)+1 {
		return false
	}
	if p := getg().m.p.ptr(); !runqempty(p) {
		return false
	}
	return true
}
```

要想实现自旋，必须符合以下四个条件

 * 自旋的次数<4 ( [`active_spin`](https://github.com/golang/go/blob/go1.16.2/src/runtime/lock_sema.go#L30))
 * CPU必须为多核处理器
 * 当前程序中设置的 `gomaxprocs` 个数 >（空闲P个数 + 当前处于自旋m的个数 + 1）
 * 至少有一个正在运行的P的本地运行队列为空

可以看到，最多 spinning 四次，超出这个次数就不再进行 spinning，以避免浪费CPU。

### 解锁 

解锁对应的方法为 ` [Unlock()](https://github.com/golang/go/blob/go1.16.2/src/sync/mutex.go#L173-L192) `，同时对应的私有方法为 ` [unlockSlow()](https://github.com/golang/go/blob/go1.16.2/src/sync/mutex.go#L194-L226) `，相比加锁代码要简单的太多了。

```
// Unlock unlocks m.
// It is a run-time error if m is not locked on entry to Unlock.
//
// A locked Mutex is not associated with a particular goroutine.
// It is allowed for one goroutine to lock a Mutex and then
// arrange for another goroutine to unlock it.
func (m *Mutex) Unlock() {
	......

	// Fast path: drop lock bit.
	new := atomic.AddInt32(&m.state, -mutexLocked)

	// 如果 new=0 表示恢复了锁的默认初始化状态，否则表示锁仍在使用
	// 解锁失败
	if new != 0 {
		// Outlined slow path to allow inlining the fast path.
		// To hide unlockSlow during tracing we skip one extra frame when tracing GoUnblock.
		m.unlockSlow(new)
	}
}
```

这里要注意一下，对于Mutex 解锁来说，在一个goroutine里加锁，在另一个goroutine是可以实现解锁的。但千万不要重复解锁，否则会触发panic。一般要遵循谁加锁，就由谁来解锁，即“ 解锁还须系铃人 ” 规则。

解锁大致流程和加锁差不多，先是执行`fast path` 原子更新，如果失败则执行 `slow path` 过程。

```
func (m *Mutex) unlockSlow(new int32) {
	// 未加锁状态，直接解锁出错
	if (new+mutexLocked)&mutexLocked == 0 {
		throw("sync: unlock of unlocked mutex")
	}

	// 正常模式 （当前m.state & mutexStarving ==0,则说明 m.state 的 mutexStarving 位是0）
	if new&mutexStarving == 0 {
		old := new
		for {
			// If there are no waiters or a goroutine has already
			// been woken or grabbed the lock, no need to wake anyone.
			// In starvation mode ownership is directly handed off from unlocking
			// goroutine to the next waiter. We are not part of this chain,
			// since we did not observe mutexStarving when we unlocked the mutex above.
			// So get off the way.

			// 如果当前队列里没有waiters 或 当前goroutine已经唤醒 或 持有了锁，则不需要唤醒其它waiter
			// 在饥饿模式下，锁控制权直接交给下一个waiter
			// 如果 当前队列没有waiter(old>>mutexWaiterShift == 0) 或 (锁为被持有状态、唤醒状态、饥饿状态其中条件之一），则直接返回
			if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken|mutexStarving) != 0 {
				return
			}
			// Grab the right to wake someone.
			// 这里 old-1<<mutexWaiterShift 表示 waiters 数量-1，mutexWoken 表示当前goroutine设置新值唤醒位值为唤醒状态，然后再通过CAS更新m.state，如果更新失败，说明当前m.state 已被其它goroutine修改过了，然后它再重新读取m.state的值，开始新一轮的for循环
			new = (old - 1<<mutexWaiterShift) | mutexWoken
			if atomic.CompareAndSwapInt32(&m.state, old, new) {
				// 抢锁控制权成功（解锁成功）
				runtime_Semrelease(&m.sema, false, 1)
				return
			}

			// 重新读取m.state值，开始新一轮判断
			old = m.state
		}
	} else {
		// Starving mode: handoff mutex ownership to the next waiter, and yield
		// our time slice so that the next waiter can start to run immediately.
		// Note: mutexLocked is not set, the waiter will set it after wakeup.
		// But mutex is still considered locked if mutexStarving is set,
		// so new coming goroutines won't acquire it.

		// 饥饿模式，将当前锁控制权直接交给下一个waiter
		runtime_Semrelease(&m.sema, true, 1)
	}
}
```

解锁源码比较好理解，对于`slow path` 而言

 * 饥饿模式直接调用函数`runtime_Semrelease()`，通过信号量将锁控制权交给下一个waiter。
 * 正常模式下分以下情况
 * 如果等待队列里没有 `waiter` 或 锁为 `被持有状态`、`唤醒状态`、`饥饿状态`三者其中条件之一，则直接返回并结束处理逻辑；
 * 当前goroutine抢锁的控制权。先读取m.state的值，waiters数量减少1，并修改状态为唤醒标记，最后通过CAS修改m.state，如果修改成功则表示抢锁控制权成功，即解锁成功，则直接结束
 * 否则重新读取m.state 的值，for 循环新一轮的逻辑

## 总结 

 * 锁模式分为`正常模式`和`饥饿模式`。正常模式下新来的goroutine与waiter竞争锁，且新来的goroutine大概率优先获取锁；饥饿模式下队列头部的waiter获取锁，新来的goroutine直接进入waiter 队列，同时也不会spinning
 * 饥饿模式下，拥有锁的waiter当发现它是队列中的最后一个waiter或者`等待时间<1毫秒`时，将自动切换为正常模式
 * 在正常模式下，新来的goroutine如果获取不到锁，则将尝试`spinning`
 * 在加锁和解锁是分 `fast path` 和 `slow path` 两种路径，加锁时执行 `runtime_SemacquireMutex()` 函数，解锁时执行对应的 `runtime_Semrelease()` 函数

## 参考 

 * [https://github.com/golang/go/blob/go1.16.2/src/sync/mutex.go#L25](https://github.com/golang/go/blob/go1.16.2/src/sync/mutex.go#L25)
 * [https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-sync-primitives/#mutex](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-sync-primitives/#mutex)
 * [https://time.geekbang.org/column/article/295850](https://time.geekbang.org/column/article/295850)
 * [https://mp.weixin.qq.com/s/lGRCaR9z4xlpU5f_ezkhzw](https://mp.weixin.qq.com/s/lGRCaR9z4xlpU5f_ezkhzw)
 * [位运算符介绍](https://blog.haohtml.com/archives/17139)