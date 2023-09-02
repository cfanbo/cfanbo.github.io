---
title: 'Runtime: Golang中channel实现原理源码分析'
author: admin
type: post
date: 2021-01-18T09:32:21+00:00
url: /archives/20760
categories:
 - 程序开发
tags:
 - channel
 - golang
 - runtime

---
channel是golang中特有的一种数据结构，通常与goroutine一起使用，下面我们就介绍一下这种数据结构。

## channel数据结构 

`channel` 是Golang 中最重要的一个数据结构，源码里对应的结构体是`hchan`，当我们创建一个`channel` 的时候，实际上是创建了一个`hchan`结构体。

### hchan结构体 

```
// src/runtime/chan.go

type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16
	closed   uint32
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}
```

字段说明

 * `qcount` 当前 channel 中的元素数量
 * `dataqsiz` 环形队列的大小
 * `buf` 指向`dataqsize`的数组指针，只有缓冲chan有效
 * `closed` 当前channel关闭状态
 * `elemsize` 存储元素的大小
 * `elemtype` 存储元素的数据类型
 * `sendx` 发送操作处理到的索引位置，最大值为数组buf的最大下标值
 * `recvx` 接收操作处理到的索引位置，最大值为数组buf的最大下标值
 * `recvq` 接收等待队列，双向链表，阻塞元素
 * `sendq` 发送等待列队，双向链表，阻塞元素
 * `lock` 锁,，用来保护sudog里的所的字段![](https://blogstatic.haohtml.com/uploads/2021/01/869afc2b8de8c84fd83c1295e930823a.png)hchan struct

其中`elemsize` 和 `elemtype` 表示存储数据的大小和类型；`sendx`和`recvx`是指向底层数据的索引位置，表示当前处理的进度位置；`recvq`和`sendq` 是一个由双向链表实现的队列，它存储的内容是由于队列`dataqsize`过小，而阻塞的数据。

每次进行发送数据和读取数据时都需要加锁。

### waitq结构体 

```
// src/runtime/chan.go

type waitq struct {
	first *sudog
	last  *sudog
}
```

### sudog结构体 

```
// src/runtime/runtime2.go

// sudog represents a g in a wait list, such as for sending/receiving
// on a channel.
//
// sudog is necessary because the g ↔ synchronization object relation
// is many-to-many. A g can be on many wait lists, so there may be
// many sudogs for one g; and many gs may be waiting on the same
// synchronization object, so there may be many sudogs for one object.
//
// sudogs are allocated from a special pool. Use acquireSudog and
// releaseSudog to allocate and free them.
type sudog struct {
	// The following fields are protected by the hchan.lock of the
	// channel this sudog is blocking on. shrinkstack depends on
	// this for sudogs involved in channel ops.

	g *g

	next *sudog
	prev *sudog
	elem unsafe.Pointer // data element (may point to stack)

	// The following fields are never accessed concurrently.
	// For channels, waitlink is only accessed by g.
	// For semaphores, all fields (including the ones above)
	// are only accessed when holding a semaRoot lock.

	acquiretime int64
	releasetime int64
	ticket      uint32

	// isSelect indicates g is participating in a select, so
	// g.selectDone must be CAS'd to win the wake-up race.
	isSelect bool

	parent   *sudog // semaRoot binary tree
	waitlink *sudog // g.waiting list or semaRoot
	waittail *sudog // semaRoot
	c        *hchan // channel
}
```

这里 `sudog` 实际上是对 `goroutine` 的一个封装，一个 sudog 就是一个goroutine，用在channal上发送和接收。

`sudogs` 是通过一个特殊的池来分配的，通过` [acquireSudog()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L321-L357) `和` [releaseSudog()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L359-L407) `进行获取和释放。

sudog里的字段是由 `hchan.lock` 锁来进行保护。

### channel 整体结构图 ![](https://blogstatic.haohtml.com/uploads/2021/01/e5efb25464faf28d6ad5c41f5d209519.png)hchan 结构图（来源： [互联网技术窝](https://mp.weixin.qq.com/s?__biz=MzUzMjk0ODI0OA==&mid=2247483766&idx=1&sn=eb605a64bed0b2066a12083f26fb04b6&chksm=faaa3501cdddbc177121ba14a6604743d5ea881ca8299d5609ac8eb9b6eca4f2a142ad5aabfd&token=1213124593&lang=zh_CN#rd))

```
// 无缓冲通道
ch1 := make(chan int)
// 有缓冲通道
ch2 := make(chan int, 10)
```

## 创建 

通过编译可以发现channel的创建是由` [makechan()](https://github.com/golang/go/blob/go1.15.6/src/runtime/chan.go#L71-L118) `函数来完成的。源码

```
// src/runtime/chan.go

func makechan(t *chantype, size int) *hchan {
	elem := t.elem

	// compiler checks this but be safe.
	if elem.size >= 1<<16 {
		throw("makechan: invalid channel element type")
	}
	if hchanSize%maxAlign != 0 || elem.align > maxAlign {
		throw("makechan: bad alignment")
	}

	mem, overflow := math.MulUintptr(elem.size, uintptr(size))
	if overflow || mem > maxAlloc-hchanSize || size < 0 {
		panic(plainError("makechan: size out of range"))
	}

	// Hchan does not contain pointers interesting for GC when elements stored in buf do not contain pointers.
	// buf points into the same allocation, elemtype is persistent.
	// SudoG's are referenced from their owning thread so they can't be collected.
	// TODO(dvyukov,rlh): Rethink when collector can move allocated objects.
	var c *hchan
	switch {
	case mem == 0:
		// Queue or element size is zero.
		c = (*hchan)(mallocgc(hchanSize, nil, true))
		// Race detector uses this location for synchronization.
		c.buf = c.raceaddr()
	case elem.ptrdata == 0:
		// Elements do not contain pointers.
		// Allocate hchan and buf in one call.
		c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
		c.buf = add(unsafe.Pointer(c), hchanSize)
	default:
		// Elements contain pointers.
		c = new(hchan)
		c.buf = mallocgc(mem, elem, true)
	}

	c.elemsize = uint16(elem.size)
	c.elemtype = elem
	c.dataqsiz = uint(size)
	lockInit(&c.lock, lockRankHchan)

	if debugChan {
		print("makechan: chan=", c, "; elemsize=", elem.size, "; dataqsiz=", size, "n")
	}
	return c
}
```

函数返回的是一个指针类型，因此我们可以在函数中通过参数直接传递，无需转为指针传递。

步骤

 1. 数据合法性检查，包括发送数据的类型和大小
 2. 根据不同场景分配内存，主要针对buf字段
 a. 内存大小为0，注意这时c.buf 的值为`c.raceaddr()`
 b. 元素不包含指针，一次性分配一段内存地址
 c. 元素包含指针，分配内存
 3. 初始化其它字段

第一个参数 *chantype 结构定义

```
// src/runtime/type.go

type chantype struct {
	typ  _type
	elem *_type
	dir  uintptr
}
```

实际上创建一个channel, 只是对一个 hchan 结构体进行了一些初始化操作，并返回其指针。因此我们在函数传递时，不需要传递指针，直接使用即可，因为它本身就是一个指针的类型。

**注意：**对于chan内存是在heap上分配的。

## 发送数据 

对于channel的写操作是由` [chansend()](https://github.com/golang/go/blob/go1.15.6/src/runtime/chan.go#L146-L284) ` 函数来实现的。

```
/*
 * generic single channel send/recv
 * If block is not nil,
 * then the protocol will not
 * sleep but return if it could
 * not complete.
 *
 * sleep can wake up with g.param == nil
 * when a channel involved in the sleep has
 * been closed.  it is easiest to loop and re-run
 * the operation; we'll see that it's now closed.
 */
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	if c == nil {
		if !block {
			return false
		}
		gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}

	...
}
```

在chan为nil的情况下, 如果是非阻塞则直接返回，否则panic。

对于发送数据 chan 有三种场景用法，分别是`直接发送，缓存区发送` 和 `阻塞发送`，其中阻塞发送涉及到GMP 的调度，理解起来有些吃力。

记得在发送数据前要进行`加锁`操作，发送完再`解锁`，保证原子性操作。

### 直接发送 

```
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	......

	// 直接发送
	// 如果接收队列中有接收者，则直接将数据发给接收者，重点在send()函数,并在函数里进行解锁
	if sg := c.recvq.dequeue(); sg != nil {
		// Found a waiting receiver. We pass the value we want to send
		// directly to the receiver, bypassing the channel buffer (if any).
		send(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true
	}

	......
}
```

如果接收队列中有接收者，则优化从接收者从队列**头部**取出一个sg(`sg := c.recvq.dequeue()`)，然后再通过调用 send() 函数将数据发送给接收者即可。![](https://blogstatic.haohtml.com/uploads/2021/01/e4b2840cddcf0a213a0f1d2d7f757e83.png)channel send

在send()函数里,会执行一个回调函数主要用来进行解锁`c.lock`。真正的发送操作是函数 ` [sendDirect()](https://github.com/golang/go/blob/go1.15.6/src/runtime/chan.go#L325-L346) `，通过` [memmove(dst, src, t.size)](https://github.com/golang/go/blob/go1.15.6/src/runtime/chan.go#L345) ` 将数据复制过去。

### 缓冲区发送 

```
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	......

	// 缓冲区发送
	// 接收者队列中没有接收者goroutine
	// 当前channel中的元素<队列的大小，有缓冲buffer未满的情况
	// 将数据存放在sendx在buf数组中的索引位置，然后再将sendx索引+1
	// 由于是一个循环数组，所以如果达到了dataqsize，则从0开始，同时个数+1
	if c.qcount < c.dataqsiz {
		// Space is available in the channel buffer. Enqueue the element to send.
		qp := chanbuf(c, c.sendx)
		if raceenabled {
			raceacquire(qp)
			racerelease(qp)
		}
		typedmemmove(c.elemtype, qp, ep)
		c.sendx++
		if c.sendx == c.dataqsiz {
			c.sendx = 0
		}
		c.qcount++
		unlock(&c.lock)
		return true
	}

	......
}
```

如果当前`recvq` 队列里没有处于等待执行的`sudog`的话，则需要将数据发送到缓冲队列中（如果当前队列为缓冲chan）。

假设当前buffer大小为6（`dataqsiz=6`），数据个数为0（`qcount=0`），这里写入6个数据，如下图。![](https://blogstatic.haohtml.com/uploads/2021/01/f1b154b7e7ba29ada72492a3538d6ac6.png)channel send

如果当前缓冲区的元素数量<队列的大小，说明缓冲区还没有满，还可以继续装载数据。

这时第一步先计算出 `s.sendx` 索引位置的内存地址，然后调用 `typememmove()` 函数将 `qp` 复制到内存地址，再将 `s.sendx` 索引值 `+1`，同时`c.qcount++`。

当 `sendx = dataqsiz` 的时候，说明已到了数组最后一个元素，下次存储数据的话，则需要重新从0开始了，所以需要重置为``。

`buf`是一个由数组组成的队列，满足队列的`FIFO`的机制，最新存储的数据优先消费，最多可以存储 `dataqsiz` 个数量。超出这个数据量就需要使用第三种 `阻塞发送` 方式了。

`sendx` 始终保存的是下次存储数据的数组索引位置，每次使用完记得`+1` 。每次存储以前都需要判断当前buffer是否有空间可用 `c.qcount < c.dataqsiz` 。

**总结**

 * `q.sendx` 最大值为 `c.dataqsiz -1`，即数组的最大索引值。
 * `q.count` 是当前chan 存储的元素个数，有可能 > `c.dataqsiz`

### 阻塞发送 

```
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	......


	// 阻塞发送
	// Block on the channel. Some receiver will complete our operation for us.
	gp := getg()
	mysg := acquireSudog()
	mysg.releasetime = 0
	if t0 != 0 {
		mysg.releasetime = -1
	}
	// No stack splits between assigning elem and enqueuing mysg
	// on gp.waiting where copystack can find it.
	mysg.elem = ep
	mysg.waitlink = nil
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.waiting = mysg
	gp.param = nil
	c.sendq.enqueue(mysg)
	// Signal to anyone trying to shrink our stack that we're about
	// to park on a channel. The window between when this G's status
	// changes and when we set gp.activeStackChans is not safe for
	// stack shrinking.
	atomic.Store8(&gp.parkingOnChan, 1)
	gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanSend, traceEvGoBlockSend, 2)
	// Ensure the value being sent is kept alive until the
	// receiver copies it out. The sudog has a pointer to the
	// stack object, but sudogs aren't considered as roots of the
	// stack tracer.
	KeepAlive(ep)

	// someone woke us up.
	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
	gp.waiting = nil
	gp.activeStackChans = false
	if gp.param == nil {
		if c.closed == 0 {
			throw("chansend: spurious wakeup")
		}
		panic(plainError("send on closed channel"))
	}
	gp.param = nil
	if mysg.releasetime > 0 {
		blockevent(mysg.releasetime-t0, 2)
	}
	mysg.c = nil
	releaseSudog(mysg)
	return true

	......
}
```

如果当buff也写满的话，再send数据的话，则需要进行阻塞发送了。![](https://blogstatic.haohtml.com/uploads/2021/01/cdd2fbd54b614576ec6b2c39457fe6cd.png)channel send

假如我们有一个缓冲chan，但缓冲大小已经使用完，再次发送数据的话，则需要进入sendq队列了（将sudog绑定到一个goroutine，并放在sendq，等待读取）

对于阻塞的情况，理解起来有些吃力，因为涉及到GMP的关系和调度。

 1. 调用 getg() 函数获取当前运行的goroutine
 2. 调用 acquireSudog() 函数获取一个sudog，并进行数据绑定
 3. 将 mysg 添加到发送队列sendq的尾部，并设置为 gp.waiting 状态
 4. 更改goroutine状态
 5. 设置goroutine为等待唤醒状态，调用 atomic.Store8(&gp.parkingOnChan, 1)函数？
 6. 通过keepAlive()函数可以保证发送的值一直有效，直到被接收者取走
 7. 进行清理工作
 8. 释放 sudog 结构体

**总结**

 * 阻塞发送并不会更新 `c.qcount` 数量个数
 * ` [acquireSudog()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L321-L357) `和 [releaseSudog(mysg)](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L360-L407) 是配对一起使用。

## 读取数据 

对于channel的读取方式:

```
v <- ch
v, ok <- ch
```

其中 v<-ch 对应的是` [runtime.chanrecv1()](https://github.com/golang/go/blob/go1.15.6/src/runtime/chan.go#L436-L440) `， v, ok <-ch 对应的是[`runtime.chanrecv2()`][1]。但这两个函数最终调用的还是同一个函数，即 ` [chanrecv()](https://github.com/golang/go/blob/go1.15.6/src/runtime/chan.go#L448-L593) `。

我们先看一下官方文档对这个函数的说明

```
// chanrecv receives on channel c and writes the received data to ep.
// ep may be nil, in which case received data is ignored.
// If block == false and no elements are available, returns (false, false).
// Otherwise, if c is closed, zeros *ep and returns (true, false).
// Otherwise, fills in *ep with an element and returns (true, true).
// A non-nil ep must point to the heap or the caller's stack.
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {}
```

 * chanrecv 用来从chan 中接收数据，并将接收的数据写入到ep
 * 如果ep为 nil 的话，则接收的数据将被忽略
 * 如果非阻塞的且没有可接收的数据将返回 (false ,false)
 * 如果chan已关闭，零值 \*ep 和返回值将是true, false，否则使用一个元素代替\*ep并返回 (true, true)
 * 一个非nil的 ep, 必须指向heap或者调用stack

```
// src/runtime/chan.go

func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
	...

	// 如果c为nil,表示非法操作，则直接gopark(),表示出让当前GMP中的P的使用权，允许其它G使用
	if c == nil {
		// 如果非阻塞的话，直接返回；如果是阻塞的话，直接panic
		if !block {
			return
		}
		gopark(nil, nil, waitReasonChanReceiveNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}
	...

	// 如果chan已关闭且元素个数为0
	if c.closed != 0 && c.qcount == 0 {
		if raceenabled {
			raceacquire(c.raceaddr())
		}
		unlock(&c.lock)
		if ep != nil {
			// 设置内存内容为类型 c.elemtype 的零值
			typedmemclr(c.elemtype, ep)
		}
		return true, false
	}

}
```

如果当前读取的 chan 为nil的话，且非阻塞的情况，则会产生死锁，最终提示

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive (nil chan)]:
```

否则返回零值。

同时出让自己占用的P，允许其它goroutine抢占使用。

如果读取的chan已关闭，则读取出来的值为零值（函数说明第四条)。

```
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
	...

	// Fast path: check for failed non-blocking operation without acquiring the lock.
	// 在没有获取锁的情况下，检查非阻塞操作失败
	if !block && empty(c) {
		// After observing that the channel is not ready for receiving, we observe whether the
		// channel is closed.
		//
		// Reordering of these checks could lead to incorrect behavior when racing with a close.
		// For example, if the channel was open and not empty, was closed, and then drained,
		// reordered reads could incorrectly indicate "open and empty". To prevent reordering,
		// we use atomic loads for both checks, and rely on emptying and closing to happen in
		// separate critical sections under the same lock.  This assumption fails when closing
		// an unbuffered channel with a blocked send, but that is an error condition anyway.

		// 如果当前chan未关闭
		if atomic.Load(&c.closed) == 0 {
			// Because a channel cannot be reopened, the later observation of the channel
			// being not closed implies that it was also not closed at the moment of the
			// first observation. We behave as if we observed the channel at that moment
			// and report that the receive cannot proceed.
			return
		}
		// The channel is irreversibly closed. Re-check whether the channel has any pending data
		// to receive, which could have arrived between the empty and closed checks above.
		// Sequential consistency is also required here, when racing with such a send.
		if empty(c) {
			// The channel is irreversibly closed and empty.
			if raceenabled {
				raceacquire(c.raceaddr())
			}
			if ep != nil {
				typedmemclr(c.elemtype, ep)
			}
			return true, false
		}
	}

	...

}
```

这段代码主要是对重复读的情况，进行了双重检测，暂时未理解 code 中考虑的情况，改天再消化消化。

```
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
	...

	var t0 int64
	if blockprofilerate > 0 {
		t0 = cputicks()
	}

	// 加锁，下面才是真正要读取的逻辑
	lock(&c.lock)

	if c.closed != 0 && c.qcount == 0 {
		if raceenabled {
			raceacquire(c.raceaddr())
		}
		unlock(&c.lock)
		if ep != nil {
			typedmemclr(c.elemtype, ep)
		}
		return true, false
	}

	...
}
```

读取之前先加锁。

对chan的读取与发送一样，同样有三种方式，为直接读取、缓冲区读取和阻塞读取。

### 直接读取 

```
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
	...

	// 直接读取
	// 从c.sendq队列中取sudog, 将数据复制到sg
	if sg := c.sendq.dequeue(); sg != nil {
		// Found a waiting sender. If buffer is size 0, receive value
		// directly from sender. Otherwise, receive from head of queue
		// and add sender's value to the tail of the queue (both map to
		// the same buffer slot because the queue is full).
		recv(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true, true
	}
}
```

获取一个待发送者，如果buffer大小为0，则直接从发送者接收数据。否则从队列**头部**接收，并将发送者发送的数据放在队列尾部。![](https://blogstatic.haohtml.com/uploads/2021/01/256e4523776ca8fdd34309d7766b6974.png)chan recv

从c.sendq队列里读取一个 *sudog，通过调用 `recv()` 函数，将数据从发送者复制到ep中，并返回true,true，表示读取成功。真正读取函数为 ` [recvDirect()](https://github.com/golang/go/blob/go1.15.6/src/runtime/chan.go#L615) `。

### 缓冲区读取 

```
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
	...

	// 如果c.qcount>0，说明缓冲区有元素可直接读取
	if c.qcount > 0 {
		// Receive directly from queue
		// 直接从队列中读取
		qp := chanbuf(c, c.recvx)
		if raceenabled {
			raceacquire(qp)
			racerelease(qp)
		}
		if ep != nil {
			typedmemmove(c.elemtype, ep, qp)
		}
		typedmemclr(c.elemtype, qp)
		c.recvx++
		if c.recvx == c.dataqsiz {
			c.recvx = 0
		}
		c.qcount--
		unlock(&c.lock)
		return true, true
	}
}
```

如果`c.qcount > 0`，则说明缓冲区里有内容可以读取。

直接获取 `c.recvx` 数组索引位置的内存地址，则

 1. 将`r.recvx` 索引地址的值读取出来复制给 ep，
 2. 然后更新接收数组索引`c.recvx++`, 如果>数组索引最大索引值 ，重置为0
 3. 减少元素总个数 `c.qcount--`
 4. 释放锁
 5. 最后unlock返回。![](https://blogstatic.haohtml.com/uploads/2021/02/3481017c61bf651f3235562780a4d16e.png)chan recv

### 阻塞读取 

```
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
	......

	// c.sendq没有sender，buffer里也是空的，直接阻塞读取
	// no sender available: block on this channel.
	gp := getg()
	mysg := acquireSudog()
	mysg.releasetime = 0
	if t0 != 0 {
		mysg.releasetime = -1
	}
	// No stack splits between assigning elem and enqueuing mysg
	// on gp.waiting where copystack can find it.
	mysg.elem = ep
	mysg.waitlink = nil
	gp.waiting = mysg
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.param = nil
	c.recvq.enqueue(mysg)
	// Signal to anyone trying to shrink our stack that we're about
	// to park on a channel. The window between when this G's status
	// changes and when we set gp.activeStackChans is not safe for
	// stack shrinking.
	atomic.Store8(&gp.parkingOnChan, 1)
	gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanReceive, traceEvGoBlockRecv, 2)

	// someone woke us up
	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
	gp.waiting = nil
	gp.activeStackChans = false
	if mysg.releasetime > 0 {
		blockevent(mysg.releasetime-t0, 2)
	}
	closed := gp.param == nil
	gp.param = nil
	mysg.c = nil
	releaseSudog(mysg)
	return true, !closed
}
```

 1. 通过 `getg()` 获取一个goroutine
 2. 调用函数 ` [acquireSudog()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L321-L357) ` 从当前P的 `sudogcache` （或 `sched.sudogcache`）中获取一个 `sudog` 结构体
 3. 绑定两者关系(将当前 g 封装在 `sudog` 结构体中)
 4. 调用 `c.recvq.enqueue(mysg)` 加入 `c.recvq` 队列的尾部
 5. 设置goroutine为等待唤醒状态
 6. 清理相关状态![](https://blogstatic.haohtml.com/uploads/2021/01/cb91c8881f18b35863281d1602f2f024.png)chan recv

## 关闭chan 

关闭chan语句

```
close(ch)
```

对于已关闭的chan，是不允许再次关闭的，否则会产生panic。对应的函数为 ` [runtime.closechan()](https://github.com/golang/go/blob/go1.15.6/src/runtime/chan.go#L357-L424) `。

```
// src/runtime/chan.go

func closechan(c *hchan) {
	// 如果chan未初始化，触发panic
	if c == nil {
		panic(plainError("close of nil channel"))
	}

	lock(&c.lock)
	// 关闭已关闭的chan，触发panicc
	if c.closed != 0 {
		unlock(&c.lock)
		panic(plainError("close of closed channel"))
	}

	......

}
```

对于一个未初始化的chan，或者已关闭的chan，如果再次关闭则会触发panic。

```
func closechan(c *hchan) {
	......
	// 设置chan关闭状态
	c.closed = 1

	// 声明一个结构体链表gList，主要用来调度使用
	var glist gList

	// release all readers
	// 释放所有readers
	for {
		sg := c.recvq.dequeue()
		if sg == nil {
			break
		}

		// 设置元素为nil
		if sg.elem != nil {
			typedmemclr(c.elemtype, sg.elem)
			sg.elem = nil
		}
		if sg.releasetime != 0 {
			sg.releasetime = cputicks()
		}
		gp := sg.g
		gp.param = nil
		if raceenabled {
			raceacquireg(gp, c.raceaddr())
		}
		glist.push(gp)
	}

	// release all writers (they will panic)
	// 释放所有writers,会引起panic,见下面说明
	for {
		sg := c.sendq.dequeue()
		if sg == nil {
			break
		}

		// 设置元素为nil
		sg.elem = nil
		if sg.releasetime != 0 {
			sg.releasetime = cputicks()
		}
		gp := sg.g
		gp.param = nil
		if raceenabled {
			raceacquireg(gp, c.raceaddr())
		}
		glist.push(gp)
	}

	// 释放锁
	unlock(&c.lock)

	// Ready all Gs now that we've dropped the channel lock.
	// 调度所有g
	for !glist.empty() {
		gp := glist.pop()
		gp.schedlink = 0
		// 唤醒goroutine
		goready(gp, 3)
	}
}
```

 1. 声明一个`gList` 链表结构体
 2. 将接收队列 `c.recvq` 中的所有元素添加到`gList` 中，并将原来的值设置为`零`值
 3. 将发送队列 `c.sendq` 中的所有元素添加到 `gList` 中，并将原来的值设置为`零`值
 4. 将所有的阻塞goroutine通过函数`goready()` 进行调度

文章里提到在对`c.sendq` 处理的时候可能会触发panic。这是因为关闭chan后，执行了 `goready()` 对原来sendq里的sudogs 进行了进行了重新调度，这时候发现chan已经关闭了，所以会panic。如这里的一个 [**例子**](https://go.dev/play/p/80ne5TQoJg9)

```
package main

import (
	"fmt"
	"time"
)

var ch chan int

func f() {

}

func main() {
	ch := make(chan int, 10)
	// buffer大小为10,这里发送11个，使最后一个进入到c.sendq里面
	for i := 0; i < 11; i++ { // i < 10 则正常
		go func(v int) {
			ch <- v
		}(i)

	}
	time.Sleep(time.Second)
	fmt.Println("发送完毕")

	// 关闭chan，将对sendq里的g进行唤醒，唤醒后发现chan关闭状态，直接panic
	close(ch)

	for v := range ch {
		fmt.Println(v)
	}
	time.Sleep(time.Second)
}
```

输出结果

```
发送完毕
4
0
panic: send on closed channel

goroutine 16 [running]:
main.main.func1(0xa)
	/tmp/sandbox2376291610/prog.go:19 +0x27
created by main.main
	/tmp/sandbox2376291610/prog.go:18 +0x45
```

多次执行可能会出现打印的数值个数不一样，但最终仍为产生 panic

有一条广泛流传的关闭 channel 的原则：

> don’t close a channel from the receiver side and don’t close a channel if the channel has multiple concurrent senders.

不要从一个 receiver 侧关闭 channel，也不要在有多个 sender 时，关闭 channel。对于只有一个sender的话，直接在sender端关闭就可以。但对于多个sender的话，则需要通过一个信号量进行关闭，参考[这里][2]。

总结

close 操作会触发goroutine的调度行为。

## 总结 

 1. 在发送和读取 chan的时候，如果chan为nil的话，这时候就根据是否阻塞进行判断是否会发生panic。如果阻塞状态的话，则会发生panic，否则会直接返回
 2. 对chan发送或接收数据的时候要保证已初始化状态
 3. 对于已关闭的chan再次关闭会触发panic
 4. 对于发送和读取数据都有三种处理情况，分别是直接读写，缓存区读写和阻塞读写
 5. 发送和接收数据的本质上是对值的`复制`操作。`All transfer of value on the go channels happens with the copy of value.`
 6. close(ch) 会触发goroutine 的被再次调度行为
 7. 内部使用 sudogs对goroutine进行了一次封装。
 8. 如果buffer中的元素无法保证消费完的话，则会产生内存泄漏的危险，这时gc是无法对这些元素时间清理的，过多的 chan就会占用大量的资源
 9. 对于chan的分配的内存是在哪里，heap还是stack?（由于管道（channel）本身是引用类型，其分配是在堆（heap）上进行的）

## 参考 

 * [https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/#)
 * [https://studygolang.com/articles/20714](https://studygolang.com/articles/20714)
 * [https://github.com/qcrao/Go-Questions/tree/master/channel](https://github.com/qcrao/Go-Questions/tree/master/channel)

 [1]: https://github.com/golang/go/blob/go1.15.6/src/runtime/chan.go#L442-L446
 [2]: https://github.com/qcrao/Go-Questions/blob/master/channel/%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E5%9C%B0%E5%85%B3%E9%97%AD%20channel.md