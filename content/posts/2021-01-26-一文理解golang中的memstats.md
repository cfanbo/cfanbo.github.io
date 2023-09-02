---
title: Golang中MemStats的介绍
author: admin
type: post
date: 2021-01-26T06:18:46+00:00
url: /archives/21685
categories:
 - 程序开发
tags:
 - golang
 - memstat

---
平时在开发中，有时间需要通过查看内存使用情况来分析程序的性能问题，经常会使用到 MemStats 这个结构体。但平时用到的都是一些最基本的方法，今天我们全面认识一下MemStas。

相关文件为 `src/runtime/mstats.go` ，本文章里主要是与内存统计相关。

## MemStats 结构体 

```
// MemStats记录有关内存分配器的统计信息
type MemStats struct {
	// General statistics.
	Alloc uint64
	TotalAlloc uint64
	Sys uint64
	Lookups uint64
	Mallocs uint64
	Frees uint64

	// Heap memory statistics.
	HeapAlloc uint64
	HeapSys uint64
	HeapIdle uint64
	HeapInuse uint64
	HeapReleased uint64
	HeapObjects uint64

	// Stack memory statistics.
	StackInuse uint64
	StackSys uint64

	// Off-heap memory statistics.
	MSpanInuse uint64
	MSpanSys uint64
	MCacheInuse uint64
	MCacheSys uint64
	BuckHashSys uint64
	GCSys uint64
	OtherSys uint64

	// Garbage collector statistics.
	NextGC uint64
	LastGC uint64
	PauseTotalNs uint64
	PauseNs [256]uint64
	PauseEnd [256]uint64
	NumGC uint32
	NumForcedGC uint32
	GCCPUFraction float64
	EnableGC bool
	DebugGC bool

	// BySize reports per-size class allocation statistics.
	BySize [61]struct {
		Size uint32
		Mallocs uint64
		Frees uint64
	}

}
```

可以清楚的看到，统计信息共分了五类

 * 常规统计信息（General statistics）
 * 分配堆内存统计（Heap memory statistics）
 * 栈内存统计（Stack memory statistics）
 * 堆外内存统计信息（Off-heap memory statistics）
 * 垃圾回收器统计信息（Garbage collector statistics）
 * 按 per-size class 大小分配统计（BySize reports per-size class allocation statistics）

以下按分类对每一个字段进行一些说明，尽量对每一个字段的用处可以联想到日常我们工作中用到的一些方法。

### 常规统计信息（General statistics） 

 * `Alloc` 已分配但尚未释放的字节
 * `TotalAlloc` 已分配（就算释放也不会减少）
 * `Sys` 系统中获取的字节(xxx_sys的统计,无锁，近似值)
 * `Nlookup` runtime执行时的指针查找数（主要在调试runtime内部使用）
 * `Nmalloc` 分配堆对象的累计数量，活动对象的数量是Mallocs-Frees
 * `Nfree` fress Frees是释放的堆对象的累计计数

### 分配堆内存统计（Heap memory statistics） 

原子更新或STW

`HeapAlloc` 已分配但尚未释放的字节(同上面的`alloc`一样)
`HeapSys` 从os为堆申请的内存大小
`HeapIdle` 空闲 spans 字节
`HeapInuse` 使用中的最大值
`HeapReleased` 操作系统的物理内存大小
`HeapObjects` 分配的堆对象总数量

### 栈内存统计（Stack memory statistics） 

stack不是heap的一部分，但runtime可以将heap内存的一部分用于stack，反之一样

`StackInuse` 在stack span的字节
`StackSys` 从os中获取的stack内存

### 堆外内存统计信息（Off-heap memory statistics） 

`MSpanInuse` 分配的mspan结构的字节
`MSpanSys` 从os中获取的用于mspan结构的字节
`MCacheInuse` 已分配的mcache结构的字节
`MCacheSys` 从os中分配的mcache结构的字节
`BuckHashSys` 分析bucket哈希表中的内存字节
`GCSys` GC中元数据的字节
`OtherSys` 其它堆外runtime分配的字节

### 垃圾回收器统计信息（Garbage collector statistics） 

`NextGC` 下次GC目标堆的大小
`LastGC` 上次GC完成的时间,UNIX时间戳
`PauseTotalNs` 从程序开始时累计暂停时长(STW), 单位纳秒
`PauseNs` 最近一次的STW时间缓存区，最近一次暂停是在 `PauseNs[(NumGC+255)%256]`，通常它是用来记录最近 `N%256` 次的GC记录。
`PauseEnd` 最近GC暂停的缓冲区，缓冲区的存放方式与PauseNs一样。每个GC有多个暂停，记录最后一次暂停
`NumGC` 完成的GC数量
`NumForcedGC` 记录应用通过调用 GC 函数强制GC的次数
`GCCPUFraction` 自程序启动后GC使用CPU时间的分值，其值为0-1之间，0表示gc没有消耗当前程序的CPU。（不包含写屏障的cpu时间）
`EnableGC` 启用GC, 值为true，除非使用GOGC=off设置
`DebugGC` 当前未使用

可以看到这些字段的信息还是比较常见的，在我们分析一个程序GC 的时候，经常会用到 `GDEBUG=gctrace=1 go run main.go` 这个命令，它的输出不正是这几个字段的信息的么？

### 按 per-size class 大小分配统计（BySize reports per-size class allocation statistics） 

目前对块概念还有些不清楚，待后期补充。

上面是对每个MemStats 结构体字段的介绍，在 mstats.go 文件中还有一个 mstat，字段与基本与MemStats一致，只是为非导出结构体，一定要保证两者的一致性，这点文件中已做了说明。

对 memstat 全局对象的操作一般是在申请或释放内存的时候发生的，如 ` [memcache.allocLarge()](https://github.com/golang/go/blob/go1.16.2/src/runtime/mcache.go#L208-L249) `、` [memcache.releaseAll()](https://github.com/golang/go/blob/go1.16.2/src/runtime/mcache.go#L251-L290) `

下面我们介绍一下与基有关的几个方法

## 使用方法 

与 `MemStats` 有关的函数只有 `ReadMemStats` 这一个。

```
// ReadMemStats populates m with memory allocator statistics.
//
// The returned memory allocator statistics are up to date as of the
// call to ReadMemStats. This is in contrast with a heap profile,
// which is a snapshot as of the most recently completed garbage
// collection cycle.
func ReadMemStats(m *MemStats) {
	stopTheWorld("read mem stats")

	systemstack(func() {
		readmemstats_m(m)
	})

	startTheWorld()
}
```

ReadMemStats 使用内存分配器统计信息填充m。

通过调用 `ReadMemStats` 返回内存分配器统计最新的信息, 与堆相反，后者是新完成GC周期的快照。

可以看到此函数会调用 ` [stopTheWorld()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L885-L918) ` 函数，也就是在收集数据期间会一直处于stop the world 状态，待收集完成后，再调用 ` [startTheWorld()](https://github.com/golang/go/blob/go1.15.6/src/runtime/proc.go#L920-L927) ` 恢复状态。

看一个例子

```
package main

import (
	"fmt"
	"runtime"
)

func main() {
	v := struct{}{}

	a := make(map[int]struct{})

	for i := 0; i < 10000; i++ {
		a[i] = v
	}

	runtime.GC()
	printMemStats("After Map Add 100000")
}

func printMemStats(mag string) {
	var m runtime.MemStats
	runtime.ReadMemStats(&m)
	fmt.Printf("%v：memory = %vKB, GC Times = %vn", mag, m.Alloc/1024, m.NumGC)
}
```