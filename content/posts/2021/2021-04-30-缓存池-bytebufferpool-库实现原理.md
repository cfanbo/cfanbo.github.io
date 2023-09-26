---
title: 缓存池 bytebufferpool 库实现原理
author: admin
type: post
date: 2021-04-30T11:48:20+00:00
url: /archives/30211
toc: true
categories:
 - 程序开发
tags:
 - golang
 - pool

---
上一节 [《Runtime: Golang 之 sync.Pool 源码分析》](https://blog.haohtml.com/archives/24697) 我们介绍了sync.Pool 的源码分析，本节介绍一个 [`fasthttp`](https://github.com/valyala/fasthttp) 中引用的一缓存池库 ` [bytebufferpool](https://github.com/valyala/bytebufferpool) `，这两个库是同一个开发者。对于这个缓存池库与同类型的几个库的对比，可以参考 [https://omgnull.github.io/go-benchmark/buffer/](https://omgnull.github.io/go-benchmark/buffer/)。

建议大家了解一下` [fasthttp](https://github.com/valyala/fasthttp) ` 这个库，性能要比直接使用内置的 `net/http` 高出很多，其主要原因是大量的用到了缓存池 `sync.Pool` 进行性能提升。

## 用法 

```go
// https://github.com/valyala/bytebufferpool/blob/18533face0/bytebuffer_example_test.go
package bytebufferpool_test

import (
	"fmt"

	"github.com/valyala/bytebufferpool"
)

func ExampleByteBuffer() {
	// 从缓存池取 Get()
	bb := bytebufferpool.Get()

	// 用法
	bb.WriteString("first linen")
	bb.Write([]byte("second linen"))
	bb.B = append(bb.B, "third linen"...)

	fmt.Printf("bytebuffer contents=%q", bb.B)

	// 使用完毕，放回缓存池 Put()
	// It is safe to release byte buffer now, since it is no longer used.
	bytebufferpool.Put(bb)
}
```

## 全局变量 

我们先看一下与其相关的一些常量

```go
const (
	// 定位数据索引位置，使用位操作性能比较高效
	minBitSize = 6 // 2**6=64 is a CPU cache line size
	// 数组索引个数 0~19
	steps      = 20

	// 最小缓存对象 和 最大缓存对象大小
	minSize = 1 << minBitSize
	maxSize = 1 << (minBitSize + steps - 1)

	// 校准阈值, 这里指的调用次数
	calibrateCallsThreshold = 42000
	// 百分比，校准数据基数
	maxPercentile           = 0.95
)
```

对于常量上面已做了注释，如果现在不明白的话没有关系，看完下面就知道它们的作用了。

## 数据类型 

主要有两个相关的数据结构，分别为 `Pool` 和 `ByteBuffer`，其实现也比较的简单。

### 数据结构 

```go
// ByteBuffer provides byte buffer, which can be used for minimizing
// memory allocations.
//
// ByteBuffer may be used with functions appending data to the given []byte
// slice. See example code for details.
//
// Use Get for obtaining an empty byte buffer.
type ByteBuffer struct {

	// B is a byte buffer to use in append-like workloads.
	// See example code for details.
	B []byte
}

// Pool represents byte buffer pool.
//
// Distinct pools may be used for distinct types of byte buffers.
// Properly determined byte buffer types with their own pools may help reducing
// memory waste.
type Pool struct {
	calls       [steps]uint64
	calibrating uint64

	defaultSize uint64
	maxSize     uint64

	pool sync.Pool
}

var defaultPool Pool
```

字段解释

 * `calls` 缓存对象大小调用次数统计，`steps` 就是我们上面定义的常量。主要用来统计每类缓存大小的调用次数。steps 具体的值会使用一个 `index()` 函数通过位操作的方式计算出来它在这个数组的索引位置；
 * `calibrating` 校标标记。0 表示未校准，1表示正在校准。校准需要一个过程，校准完成后需要从1恢复为 0;
 * `defaultSize` 缓存对象默认大小。我们知道当从 pool 中获取缓存对象时，如果池中没有对象可取，会通过调用 一个 `New()` 函数创建一个新对象返回，这时新创建的对象大小为 `defaultSize`。当然这里没有使用 `New()` 函数,而是直接创建了一个 指定默认大小的 `ByteBuffer`；
 * `maxSize` 允许放入`pool`池中的最大对象大小，只有`

## 实现原理 

```go
// Get returns an empty byte buffer from the pool.
func Get() *ByteBuffer { return defaultPool.Get() }

// Put returns byte buffer to the pool.
func Put(b *ByteBuffer) { defaultPool.Put(b) }

```

这里有两个全局函数，分别为 ` [Get()](https://github.com/valyala/bytebufferpool/blob/master/pool.go#L37-L42) ` 和 ` [Put()](https://github.com/valyala/bytebufferpool/blob/master/pool.go#L58-L62) ` ，很容易理解就是一个取缓存和存缓存的函数。函数里调用的是全局变量 `defaultPool` 相应的方法。

### 取对象 

对于取对象很简单

```go
// Get returns new byte buffer with zero length.
//
// The byte buffer may be returned to the pool via Put after the use
// in order to minimize GC overhead.
func (p *Pool) Get() *ByteBuffer {
	v := p.pool.Get()
	if v != nil {
		return v.(*ByteBuffer)
	}
	return &ByteBuffer{
		B: make([]byte, 0, atomic.LoadUint64(&p.defaultSize)),
	}
}
```

操作步骤

 * 直接从 `p.pool` 中调用原始方法 `Get()` 读取，如果结果 `!= nil`，则说明当前池中读取到了数据（当pool未设置New 方法的时候会返回nil）,则直接返回对象 `*ByteBuffer` 即可；
 * 如果结果等于 `nil` ，则说明池中已无对象可用且未定义New方法，这时直接创建一个 `p.defaultSize` 大小的 `*ByteBuffer` 对象并返回

对于从 `pool` 池中取对象的顺序依次为 `pool.local.poolLocal.poolLocalInternal.private` -> `pool.local.poolLocal.poolLocalInternal.share` -> `调用getSlow()从其它P的 share 尾部窃取` -> `调用 New() 方法创建`，如果最后没有定义`New()`，则直接返回 `nil`。

### 存对象 

存对象稍微有一点点复杂，主要是多了一个校准的操作。

```go
// Put releases byte buffer obtained via Get to the pool.
//
// The buffer mustn't be accessed after returning to the pool.
func (p *Pool) Put(b *ByteBuffer) {
	// 对象在数据中的位置
	idx := index(len(b.B))

	// 校准条件
	if atomic.AddUint64(&p.calls[idx], 1) > calibrateCallsThreshold {
		p.calibrate()
	}

	// 是否需要放入pool池中，还是直接丢弃交给GC
	maxSize := int(atomic.LoadUint64(&p.maxSize))
	if maxSize == 0 || cap(b.B) <= maxSize {
		b.Reset()
		p.pool.Put(b)
	}
}
```

操作步骤

 1. 调用 ` [index()](https://github.com/valyala/bytebufferpool/blob/master/pool.go#L139-L151) ` 函数，根据对象长度计算其在 `p.calls` 数组中的索引位置;
 2. 将当前数组索引位置存放的值原子操作+1，（即更新相同位置对象的调用次数，注意：更新操作并没有放在Get）,如果 `次数>calibrateCallsThreshold`（42000） ，则进行校准操作;
 3. 原子读取当前允许放入pool池中的对象大小。如果等于0或小于 `maxSize` ，则先 `b.Reset()` 重置对象,再将其放入pool池中; 否则将交由GC 来操作；

上面第三步在放入池中的时候，为什么这里还要判断大小才能放回pool池中呢？

主要原因是因为这里用的是一个 `切片` 数据类型，虽然在放入前执行了`b.Reset`，但这只是将切片里的内容进行了清除，但这个切片对象仍然是处于引用状态，并没有真正释放内存。如果一个对象从pool 取出来以后，经过一系列操作后，导致这个切片非常非常的大，这时再将其放入pool池中，此切片会一直占用着很大一块的内部，导致内存泄漏。

整体逻辑还是比较清楚的。下面我们重点看一下 `校准` 逻辑

```go
func (p *Pool) calibrate() {
	if !atomic.CompareAndSwapUint64(&p.calibrating, 0, 1) {
		return
	}

	a := make(callSizes, 0, steps)
	var callsSum uint64
	for i := uint64(0); i < steps; i++ {
		calls := atomic.SwapUint64(&p.calls[i], 0)
		callsSum += calls
		a = append(a, callSize{
			calls: calls,
			size:  minSize << i,
		})
	}
	sort.Sort(a)

	defaultSize := a[0].size
	maxSize := defaultSize

	maxSum := uint64(float64(callsSum) * maxPercentile)
	callsSum = 0
	for i := 0; i < steps; i++ {
		if callsSum > maxSum {
			break
		}
		callsSum += a[i].calls
		size := a[i].size
		if size > maxSize {
			maxSize = size
		}
	}

	atomic.StoreUint64(&p.defaultSize, defaultSize)
	atomic.StoreUint64(&p.maxSize, maxSize)

	atomic.StoreUint64(&p.calibrating, 0)
}
```

先判断当前是否处于正在校准状态，如果是则直接终止，否则继续执行。

先声明一个长度为`20(steps)` 的 ` [callSize](https://github.com/valyala/bytebufferpool/blob/master/pool.go#L120-L123) ` 切片类型变量 `callSizes` ，然后将 `p.calls` 中统计的调用次数依次放入对象 `callSize.calls` 中，同时根据其索引位置计算对应存放对象的大小(`minSize<https://github.com/valyala/bytebufferpool
 * [https://blog.haohtml.com/archives/24697](https://blog.haohtml.com/archives/24697)