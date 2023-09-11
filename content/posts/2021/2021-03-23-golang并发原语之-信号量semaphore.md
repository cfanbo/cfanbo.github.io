---
title: Golang并发同步原语之-信号量Semaphore
author: admin
type: post
date: 2021-03-23T13:14:21+00:00
url: /archives/25563
categories:
 - 程序开发
tags:
 - golang
 - Semaphore
 - 并发原语

---
信号量是并发编程中比较常见的一种同步机制，它会保持资源计数器一直在`0-N`（`N`表示权重值大小，在用户初始化时指定）之间。当用户获取的时候会减少一点，使用完毕后再恢复过来。当遇到请求时资源不够的情况下，将会进入休眠状态以等待其它进程释放资源。

在 Golang 官方扩展库中为我们提供了一个基于权重的信号量 ` [semaphore](https://github.com/golang/sync/blob/master/semaphore/semaphore.go) ` 并发原语。

你可以将下面的参数 `n` 理解为资源权重总和，表示每次获取时的权重；也可以理解为资源数量，表示每次获取时必须一次性获取的资源数量。为了理解方便，这里直接将其理解为资源数量。

## 数据结构 

` [semaphoreWeighted](https://github.com/golang/sync/blob/master/semaphore/semaphore.go#L19-L33) ` 结构体

```
type waiter struct {
	n     int64
	ready chan<- struct{} // Closed when semaphore acquired.
}

// NewWeighted creates a new weighted semaphore with the given
// maximum combined weight for concurrent access.
func NewWeighted(n int64) *Weighted {
	w := &Weighted{size: n}
	return w
}

// Weighted provides a way to bound concurrent access to a resource.
// The callers can request access with a given weight.
type Weighted struct {
	size    int64
	cur     int64
	mu      sync.Mutex
	waiters list.List
}
```

一个 `watier` 就表示一个请求，其中n表示这次请求的资源数量（权重）。

使用 `NewWeighted()` 函数创建一个并发访问的最大资源数，这里 `n` 表示资源个数。

`Weighted` 字段说明

 * `size` 表示最大资源数量，取走时会减少，释放时会增加
 * `cur` 计数器，记录当前已使用资源数，值范围`[0 - size]`
 * `mu` 锁
 * `waiters` 当前处于等待休眠的请求者`goroutine`，每个请求者请求的资源数量可能不一样，只有在请求时，可用资源数量不足时请求者才会进入请求链表，每个请求者表示一个`goroutine`

计数器 `cur` 会随着资源的获取和释放而变化，那么为什么要引入数量（权重）这个概念呢？

## 方法列表 

 * [type Weighted][1]
 * [func NewWeighted(n int64) *Weighted][2]
 * [func (s *Weighted) Acquire(ctx context.Context, n int64) error][3]
 * [func (s *Weighted) Release(n int64)][4]
 * [func (s *Weighted) TryAcquire(n int64) bool][5]

方法

 * `NewWighted` 方法用来创建一类资源，参数 `n` 资源表示最大可用资源总个数;
 * `Acquire` 获取指定个数的资源，如果当前没有空闲资源可用，当前请求者goroutine将陷入休眠状态;
 * `Release` 释放资源
 * `TryAcquire` 同 `Acquire` 一样，但当无空闲资源将直接返回`false`，而不阻塞。

## 获取 Acquire 和 TryAcquire 

对于获取资源有两种方法，分别为 ` [Acquire()](https://github.com/golang/sync/blob/master/semaphore/semaphore.go#L35-L83) ` 和 ` [TryAcquire()](https://github.com/golang/sync/blob/master/semaphore/semaphore.go#L85-L95) `，两者的区别我们上面已介绍过。

在获取和释放资源前必须先加`全局锁`。

获取资源时根据空闲资源情况，可分为三种：

 * 有空闲资源可用，将返回`nil`，表示成功
 * 请求资源数量超出了初始化时指定的总数量，这个肯定永远也不可能执行成功的，所以直接返回 `ctx.Err()`
 * 当前空闲资源数量不足，需要等待其它goroutine对资源进行释放才可以运行，这时将当前请求者goroutine放入等待队列。 这里再根据情况而定，具体见 select 判断

```
// Acquire acquires the semaphore with a weight of n, blocking until resources
// are available or ctx is done. On success, returns nil. On failure, returns
// ctx.Err() and leaves the semaphore unchanged.
//
// If ctx is already done, Acquire may still succeed without blocking.
func (s *Weighted) Acquire(ctx context.Context, n int64) error {
	// 有可用资源，直接成功返回nil
	s.mu.Lock()
	if s.size-s.cur >= n && s.waiters.Len() == 0 {
		s.cur += n
		s.mu.Unlock()
		return nil
	}

	// 请求资源权重远远超出了设置的最大权重和，失败返回 ctx.Err()
	if n > s.size {
		// Don't make other Acquire calls block on one that's doomed to fail.
		s.mu.Unlock()
		<-ctx.Done()
		return ctx.Err()
	}

	// 有部分资源可用，将请求者放在等待队列(头部），并通过select 实现通知其它waiters
	ready := make(chan struct{})
	w := waiter{n: n, ready: ready}
	// 放入链表尾部，并返回放入的元素
	elem := s.waiters.PushBack(w)
	s.mu.Unlock()

	select {
	case <-ctx.Done():
		// 收到外面的控制信号
		err := ctx.Err()
		s.mu.Lock()
		select {
		case <-ready:
			// Acquired the semaphore after we were canceled.  Rather than trying to
			// fix up the queue, just pretend we didn't notice the cancelation.
			// 如果在用户取消之前已经获取了资源,则直接忽略这个信号，返回nil表示成功
			err = nil
		default:
			// 收到控制信息，且还没有获取到资源，就直接将原来添加的 waiter 删除
			isFront := s.waiters.Front() == elem

			// 则将其从链接删除 上面 ctx.Done()
			s.waiters.Remove(elem)

			// 如果当前元素正好位于链表最前面，且还存在可用的资源，就通知其它waiters
			if isFront && s.size > s.cur {
				s.notifyWaiters()
			}
		}
		s.mu.Unlock()
		return err

	case <-ready:
		return nil
	}
}
```

注意上面在`select`逻辑语句上面有一次加解锁的操作，在 `select` 里面由于是全局锁所以还需要再次加锁。

根据可用计数器信息，可分三种情况：

 1. 对于 ` [TryAcquire()](https://github.com/golang/sync/blob/master/semaphore/semaphore.go#L85-L95) ` 就比较简单了，就是一个可用资源数量的判断，数量够用表示成功返回 `true` ，否则 `false`，此方法并不会进行阻塞，而是直接返回。

```
// TryAcquire acquires the semaphore with a weight of n without blocking.
// On success, returns true. On failure, returns false and leaves the semaphore unchanged.
func (s *Weighted) TryAcquire(n int64) bool {
	s.mu.Lock()
	success := s.size-s.cur >= n && s.waiters.Len() == 0
	if success {
		s.cur += n
	}
	s.mu.Unlock()
	return success
}
```

## 释放 Release 

对于释放也很简单，就是将已使用资源数量（计数器）进行更新减少，并通知其它 `waiters`。

```
// Release releases the semaphore with a weight of n.
func (s *Weighted) Release(n int64) {
	s.mu.Lock()
	s.cur -= n
	if s.cur < 0 {
		s.mu.Unlock()
		panic("semaphore: released more than held")
	}
	s.notifyWaiters()
	s.mu.Unlock()
}
```

## 通知机制 

通过 `for` 循环从链表头部开始依次遍历出链表中的所有`waiter`，并更新计数器 `Weighted.cur`，同时将其从链表中删除，直到遇到 `空闲资源数量 < watier.n` 为止。

```
func (s *Weighted) notifyWaiters() {
	for {
		next := s.waiters.Front()
		if next == nil {
			break // No more waiters blocked.
		}

		w := next.Value.(waiter)
		if s.size-s.cur < w.n {
			// Not enough tokens for the next waiter.  We could keep going (to try to
			// find a waiter with a smaller request), but under load that could cause
			// starvation for large requests; instead, we leave all remaining waiters
			// blocked.
			//
			// Consider a semaphore used as a read-write lock, with N tokens, N
			// readers, and one writer.  Each reader can Acquire(1) to obtain a read
			// lock.  The writer can Acquire(N) to obtain a write lock, excluding all
			// of the readers.  If we allow the readers to jump ahead in the queue,
			// the writer will starve — there is always one token available for every
			// reader.
			break
		}

		s.cur += w.n
		s.waiters.Remove(next)
		close(w.ready)
	}
}
```

可以看到如果一个链表里有多个等待者，其中一个等待者需要的资源（权重）比较多的时候，当前 `watier` 会出现长时间的阻塞（即使当前可用资源足够其它`waiter`执行，期间会有一些资源浪费）， 直到有足够的资源可以让这个等待者执行，然后继续执行它后面的等待者。

## 使用示例 

官方文档提供了一个基于信号量的典型的“`工作池`”模式，见 [https://pkg.go.dev/golang.org/x/sync/semaphore#example-package-WorkerPool](https://pkg.go.dev/golang.org/x/sync/semaphore#example-package-WorkerPool)，演示了如何通过信号量控制一定数量的 `goroutine` 并发工作。

这是一个通过信号量实现并发` [考拉兹猜想](https://zh.wikipedia.org/wiki/%E8%80%83%E6%8B%89%E5%85%B9%E7%8C%9C%E6%83%B3) ` 的示例，对`1-32`之间的数字进行计算，并打印32个符合结果的值。

```
package main

import (
	"context"
	"fmt"
	"log"
	"runtime"

	"golang.org/x/sync/semaphore"
)

// Example_workerPool demonstrates how to use a semaphore to limit the number of
// goroutines working on parallel tasks.
//
// This use of a semaphore mimics a typical “worker pool” pattern, but without
// the need to explicitly shut down idle workers when the work is done.
func main() {
	ctx := context.TODO()

 	// 权重值为逻辑cpu个数
	var (
		maxWorkers = runtime.GOMAXPROCS(0)
		sem        = semaphore.NewWeighted(int64(maxWorkers))
		out        = make([]int, 32)
	)

	// Compute the output using up to maxWorkers goroutines at a time.
	for i := range out {
		// When maxWorkers goroutines are in flight, Acquire blocks until one of the
		// workers finishes.
		if err := sem.Acquire(ctx, 1); err != nil {
			log.Printf("Failed to acquire semaphore: %v", err)
			break
		}

		go func(i int) {
			defer sem.Release(1)
			out[i] = collatzSteps(i + 1)
		}(i)
	}

	// 如果使用了 errgroup 原语则不需要下面这段语句
	if err := sem.Acquire(ctx, int64(maxWorkers)); err != nil {
		log.Printf("Failed to acquire semaphore: %v", err)
	}

	fmt.Println(out)

}

// collatzSteps computes the number of steps to reach 1 under the Collatz
// conjecture. (See https://en.wikipedia.org/wiki/Collatz_conjecture.)
func collatzSteps(n int) (steps int) {
	if n <= 0 {
		panic("nonpositive input")
	}

	for ; n > 1; steps++ {
		if steps < 0 {
			panic("too many steps")
		}

		if n%2 == 0 {
			n /= 2
			continue
		}

		const maxInt = int(^uint(0) >> 1)
		if n > (maxInt-1)/3 {
			panic("overflow")
		}
		n = 3*n + 1
	}

	return steps
}
```

上面先声明了总权重值为逻辑CPU数量，每次 `for` 循环都会调用一次 `sem.Acquire(ctx, 1)`， 即表示最多每个CPU可运行一个 goroutine，如果当前权重值不足的话，其它groutine将处于阻塞状态，这里共循环32次，即阻塞数量最大为 `32-maxWorkers`。

每获取成功一个权重就会执行go匿名函数，并在函数结束时释放权重。为了保证每次for循环都会正常结束，最后调用了 `sem.Acquire(ctx, int64(maxWorkers))` ，表示最后一次执行必须获取的权重值为 `maxWorkers`。当然如果使用 `errgroup` 同步原语的话，这一步可以省略掉

以下为使用 `errgroup` 的方法

```
func main() {
	ctx := context.TODO()
	var (
		maxWorkers = runtime.GOMAXPROCS(0)
		sem        = semaphore.NewWeighted(int64(maxWorkers))
		out        = make([]int, 32)
	)

	group, _ := errgroup.WithContext(context.Background())
	for i := range out {
		if err := sem.Acquire(ctx, 1); err != nil {
			log.Printf("Failed to acquire semaphore: %v", err)
			break
		}
		group.Go(func() error {
			go func(i int) {
				defer sem.Release(1)
				out[i] = collatzSteps(i + 1)
			}(i)
			return nil
		})
	}

	// 这里会阻塞，直到所有goroutine都执行完毕
	if err := group.Wait(); err != nil {
		fmt.Println(err)
	}
	fmt.Println(out)
}
```

**推荐** [Golang中的并发原语 Singleflight](https://blog.haohtml.com/archives/20279)

 [1]: https://pkg.go.dev/golang.org/x/sync/semaphore#Weighted
 [2]: https://pkg.go.dev/golang.org/x/sync/semaphore#NewWeighted
 [3]: https://pkg.go.dev/golang.org/x/sync/semaphore#Weighted.Acquire
 [4]: https://pkg.go.dev/golang.org/x/sync/semaphore#Weighted.Release
 [5]: https://pkg.go.dev/golang.org/x/sync/semaphore#Weighted.TryAcquire