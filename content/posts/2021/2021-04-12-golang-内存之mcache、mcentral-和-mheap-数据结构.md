---
title: Golang 内存组件之mspan、mcache、mcentral 和 mheap 数据结构
author: admin
type: post
date: 2021-04-12T09:42:59+00:00
url: /archives/29385
toc: true
categories:
 - 程序开发
tags:
 - golang
 - mcache
 - mspan

---
Golang中的内存组件关系如下图所示![components of memory allocation](https://blogstatic.haohtml.com/uploads/2021/04/5a666325bb7cfea6f5182e0ee7c528cf.jpg)golang 内存分配组件

在学习golang 内存时，经常会涉及几个重要的数据结构，如果不熟悉它们的情况下，理解起来就显得格外的吃力，所以本篇主要对相关的几个内存组件做下数据结构的介绍。

在 Golang 中，`mcache`、`mspan`、`mcentral` 和 `mheap` 是内存管理的四大组件，`mcache` 管理线程在本地缓存的 `mspan`，而 `mcentral` 管理着全局的 `mspan` 为所有 `mcache` 提供所有线程。

根据分配对象的大小，内部会使用不同的内存分配机制，详细参考函数 ` [mallocgo()](https://github.com/golang/go/blob/go1.16.2/src/runtime/malloc.go#L902-L1171) `，所于内存分配与回收，参考文件介绍 [malloc.go](https://github.com/golang/go/blob/go1.15.6/src/runtime/malloc.go#L5)

 * `<16KB` 会使用微小对象内存分配器从 `P` 中的 `mcache` 分配，主要使用 `mcache.tinyXXX` 这类的字段
 * `16-32KB` 从 `P` 中的 `mcache` 中分配
 * `>32KB` 直接从 `mheap` 中分配

对于golang中的内存申请流程，大家应该都非常熟悉了，这里不再进行详细描述。![](https://blogstatic.haohtml.com/uploads/2021/04/1bb7fe2168b7ac2e24afadf698dc6ee6.png)Golang 内存组件关系

# mcache 

在GPM关系中，会在每个 `P` 下都有一个 `mcache` 字段，用来表示内存信息。

在 Go 1.2 版本以前调度器使用的是 `GM` 模型，将 `mcache` 放在了 `M` 里，但发现存在诸多问题，其中对于内存这一块存在着巨大的浪费。每个 `M` 都持有 `mcache` 和 `stack alloc`，但只有在 `M` 运行 Go 代码时才需要使用内存(每个 mcache 可以高达2mb)，当 `M` 在处于 `syscall` 或 `网络请求` 的时候是不需要内存的，再加上 `M` 又是允许创建多个的，这就造成了内存的很大浪费。所以从go 1.3版本开始使用了GPM模型，这样在高并发状态下，每个G只有在运行的时候才会使用到内存，而每个 G 会绑定一个P，所以它们在运行时只占用一份 mcache，对于 mcache 的数量就是P 的数量，同时并发访问时也不会产生锁。

对于 GM 模型除了上面提供到内存浪费的问题，还有其它问题，如单一全局锁 sched.Lock、goroutine 传递问题和内存局部性等等。

在 `P` 中，一个 `mcache` 除了可以用来缓存小对象外，还包含一些本地分配统计信息。由于在每个P下面都存在一个`mcache` ，所以多个 `goroutine` (每个P运行一个g)并发请求内存时是无锁的。

![](https://blogstatic.haohtml.com/uploads/2021/04/d2b5ca33bd970f64a6301fa75ae2eb22-2.png)mcache

当申请一个 `16k` 大小的内存时，会优先从运行当前 G 所在的 `P` 里的 `mcache` 字段里找到相匹配的 `mspan` 规格，此时是不需要锁的，这里最合适的是图中 `mspan3` 规格。

> 上图中的16b，这里的 b 是指 bytes，并不是 bit

mcache是从非GC内存中分配的，所以任何一个堆指针都必须经过特殊处理。源码文件： [https://github.com/golang/go/blob/go1.16.2/src/runtime/mcache.go](https://github.com/golang/go/blob/go1.16.2/src/runtime/mcache.go)

```
type mcache struct {
	// 下方成员会在每次访问malloc时都会被访问，所以为了更加高效缓存将其按组放在这里
	nextSample uintptr // trigger heap sample after allocating this many bytes
	scanAlloc  uintptr // bytes of scannable heap allocated

	// 小对象缓存，<16b。推荐阅读"Tiny allocator"注释文档
	tiny       uintptr
	tinyoffset uintptr
	tinyAllocs uintptr

	// 下方成员不会在每次 malloc 时被访问
	alloc [numSpanClasses]*mspan // spans to allocate from, indexed by spanClass

	stackcache [_NumStackOrders]stackfreelist

	flushGen uint32
}
```

 * `nextSample` 分配多少大小的堆时触发堆采样;
 * `scannAlloc` 分配的可扫描堆字节数;
 * `tiny` 堆指针，指向当前 `tiny` 块的起始指针，如果当前无tiny块则为`nil`。在终止标记期间，通过调用 `mcache.releaseAll()` 来清除它;
 * `tinyoffset` 当前tiny 块的位置;
 * `tinyAllocs` 拥有当前 `mcache` 的 P 执行的微小分配数;
 * `alloc [numSpanClasses]*mspan` 当前P的分配规格信息，共 `numSpanClasses = _NumSizeClasses << 1` 种规格，每个规格的都存在两份，一个包含指针，另一个不包含指针，GC时只对包含指针的这份进行处理
 * `stackcache` 内存规格序号，按 `spanClass` 索引，参考 [这里](https://github.com/golang/go/blob/go1.16.2/src/runtime/malloc.go#L153);
 * `flushGen` 表示上次刷新 `mcache` 的 `sweepgen`（清扫生成）。如果 `flushGen != mheap_.sweepgen` 则说明 `mcache` 已过期需要刷新，需被清扫。在 `acrequirep` 中完成;

`mcache.tiny` 是一个指针，当申请对象大小为 `<16KB` 的时候，会使用 `Tiny allocator` 分配器，会根据`tiny`、`tinyoffset` 和 `tinyAllocs` 这三个字段的情况进行申请。

`span` 大小规格数据共有 `67` 类。源码里定义的虽然是 ` [_NumSizeClasses = 68](https://github.com/golang/go/blob/go1.16.2/src/runtime/sizeclasses.go#L80) ` 类，但其中包含一个大小为 `0` 的规格，此规格表示大对象，即 `>32KB`，这种对象只会分配到`heap`上，所以不可能出现在 `mcache.alloc` 中。

`mcache.alloc` 是一个数组，值为 `*spans` 类型，它是 go 中管理内存的基本单元。对于`16-32 kb`大小的内存都会使用这个数组里的的 `spans` 中分配。每个span存在两次，一个`不包含指针`的对象列表和另一个`包含指针`的对象列表。这种区别将使垃圾收集工作变得更容易，因为它不必扫描不包含任何指针的范围。

# mspan 

`mspan` 是分配内存时的基本单元。当分配内存时，会在`mcache`中查找合适规格的可用 `mspan`，此时不需要加锁，因此分配效率极高。

Go将内存块分为大小不同的 `67` 种，然后再把这 `67` 种大内存块，逐个分为小块(可以近似理解为大小不同的相当于`page`)称之为`span`(连续的`page`)

> 经常提到的span和mspan其实是同一个东西，在runtime数据结构里span对应的实现结构体为 mspan

![6328562-c86d915ad1df4bbb](https://blogstatic.haohtml.com/uploads/2021/04/c3a0528cf9a2e30f31aab3a5c9066af2-24.png)mspans

对象分配的时候，根据对象的大小选择能容纳其大小的最小`span`。

`spans` 与 `mcache` 的关系如下图所示![20210129154022](https://blogstatic.haohtml.com/uploads/2021/04/87f2fdbbff345ac3f85fdd3135b1b397-3.png)mspans

```
// mSpanList heads a linked list of spans.
// 指向spans链表
//go:notinheap
type mSpanList struct {
	first *mspan // first span in list, or nil if none
	last  *mspan // last span in list, or nil if none
}

//go:notinheap
type mspan struct {
	next *mspan     // next span in list, or nil if none
	prev *mspan     // previous span in list, or nil if none
	list *mSpanList // For debugging. TODO: Remove.

	startAddr uintptr // address of first byte of span aka s.base()
	npages    uintptr // number of pages in span

	manualFreeList gclinkptr // list of free objects in mSpanManual spans

	freeindex uintptr

	nelems uintptr // number of object in the span.

	allocCache uint64

	allocBits  *gcBits
	gcmarkBits *gcBits

	// sweep generation:
	// if sweepgen == h->sweepgen - 2, the span needs sweeping
	// if sweepgen == h->sweepgen - 1, the span is currently being swept
	// if sweepgen == h->sweepgen, the span is swept and ready to use
	// if sweepgen == h->sweepgen + 1, the span was cached before sweep began and is still cached, and needs sweeping
	// if sweepgen == h->sweepgen + 3, the span was swept and then cached and is still cached
	// h->sweepgen is incremented by 2 after every GC

	sweepgen    uint32
	divMul      uint16        // for divide by elemsize - divMagic.mul
	baseMask    uint16        // if non-0, elemsize is a power of 2, & this will get object allocation base
	allocCount  uint16        // number of allocated objects
	spanclass   spanClass     // size class and noscan (uint8)
	state       mSpanStateBox // mSpanInUse etc; accessed atomically (get/set methods)
	needzero    uint8         // needs to be zeroed before allocation
	divShift    uint8         // for divide by elemsize - divMagic.shift
	divShift2   uint8         // for divide by elemsize - divMagic.shift2
	elemsize    uintptr       // computed from sizeclass or from npages
	limit       uintptr       // end of data in span
	speciallock mutex         // guards specials list
	specials    *special      // linked list of special records sorted by offset.
}
```

`mSpanList` 是一个`mspans`链表，这个很好理解。重点看下 mspan 结构体

 * `next` 指向下一个 `span` 的指针，为 `nil` 表示没有
 * `prev` 指向上一个 `span` 的指针，与 `next` 相反
 * `list` 指向 `mSpanList`，调试使用，以后会废弃
 * `startAddr` `span`第一个字节地址，可通过 `s.base()` 函数读取
 * `npages` span中的页数（一个`span` 是由多个`page`组成的，与linux中的页不是同一个概念）
 * `manualFreeList` 在 `mSpanManual` spans中的空闲对象的列表
 * `freeindex` 标记 `0~nelems` 之间的插槽索引，标记的是在`span`中的分配内存时起始扫描位置;
 每次分配内存都从 `allocBits` 的 `freeindex` 索引位置开始，直到遇到 `0` ,表示空闲对象，然后调整 `freeindex` 使得下一次扫描能跳过上一次的分配；
  若 `freeindex==nelem`，则当前`span`没有了空余对象；
  `allocBits` 是对象在 `span` 中的位图；
  如果 `n >= freeindex and allocBits[n/8] & (1<<(n%8)) == 0` , 那么对象 `n` 是空闲的；
  否则，对象 `n` 表示已被分配。从 `elem` 开始的是未定义的，将不应该被定义；
 * `nelems` span中对象数（`page`是内存存储的基本单元, 一个`span`由多个`page`组成，同时一个对象可能占用一个或多个`page`)；
 * `allocCache` 在 `freeindex` 位置的 `allocBits` 缓存；
 * `allocBits` 标记span中的elem哪些是“被使用”了的，哪些是未被使用的；清除后将释放 `allocBits` ，并将 `allocBits` 的值设置为 `gcmarkBits`；
 * `gcmarkBits` 标记span中的elem哪些是“被标记”了的，哪些是未被标记的；
 * `spanclass` spanClass类型；
 * `state` 由于协程栈也是从堆上分配的，也在mheap管理的这些span中，`mspan.spanState`会记录该span是用作堆内存，还是用作栈内存；![](https://blogstatic.haohtml.com/uploads/2022/05/94ef332c39e0b9f5a853cc29bc9349a7.png)

每个 `mspan` 都对应两个位图标记：`mspan.allocBits` 和 `mspan.gcmarkBits`。

**（1）allocBits**中每一位用于标记一个对象存储单元是否已分配。![](https://blogstatic.haohtml.com/uploads/2021/04/d2b5ca33bd970f64a6301fa75ae2eb22-4.png)allocBits

**（2）gcmarkBits**中每一位用于标记一个对象是否存活。![](https://blogstatic.haohtml.com/uploads/2021/04/d2b5ca33bd970f64a6301fa75ae2eb22-5.png)gcMarkBits

**03. Golang中GC的三色标记**

 * （1）着为灰色对应的操作就是把指针对应的`gcmarkBits`标记位置为 `1`并加入工作队列。
 * （2）着为黑色对应的操作就是把对象对应的`gcmarkBits`标记位置为`1`。
 * （3）白色对象就是那些`gcmarkBits`中标记为 `0` 的对象。

# mcentral 

`mentral` 是一个空闲列表。

实际上 `mcentral` 它并不包含空闲对象列表，真正包含的是 `mspan` 。

每个`mcentral` 是两个 `mspans` 列表：空闲对象 `c->notempty` 和 完全分配对象 `c->empty`，如图所示![](https://blogstatic.haohtml.com/uploads/2021/04/d2b5ca33bd970f64a6301fa75ae2eb22-1.png)mcentral

当申请一个 `16b` 大小的内存时，如果 `p.mcache` 中无可用大小内存时，则它找一个最合适的规则 `mcentral` 查找，如图所示这时会在存放`16b`大小的 `mcentral` 中的 `notempty` 里查找。

文件源码： [https://github.com/golang/go/blob/go1.16.2/src/runtime/mcentral.go](https://github.com/golang/go/blob/go1.16.2/src/runtime/mcentral.go)

```
type mcentral struct {
	spanclass spanClass
	partial [2]spanSet // list of spans with a free object
	full    [2]spanSet // list of spans with no free objects
}
```

 * `spanClass` 指当前规格大小
 * `partial` 存在空闲对象spans列表
 * `full` 无空闲对象spans列表

其中 `partial` 和 `full` 都包含两个 `spans` 集数组。一个用在扫描 spans,另一个用在未扫描spans。在每轮GC期间都扮演着不同的角色。`mheap_.sweepgen` 在每轮gc期间都会递增2。

`partial` 和 `full` 的数据类型为 `spanSet`，表示 `*mspans` 集。

```
type spanSet struct {
	spineLock mutex
	spine     unsafe.Pointer // *[N]*spanSetBlock, accessed atomically
	spineLen  uintptr        // Spine array length, accessed atomically
	spineCap  uintptr        // Spine array cap, accessed under lock

	index headTailIndex
}
```

对 `mcentral` 的初始化如下

```
// Initialize a single central free list.
func (c *mcentral) init(spc spanClass) {
	c.spanclass = spc
	lockInit(&c.partial[0].spineLock, lockRankSpanSetSpine)
	lockInit(&c.partial[1].spineLock, lockRankSpanSetSpine)
	lockInit(&c.full[0].spineLock, lockRankSpanSetSpine)
	lockInit(&c.full[1].spineLock, lockRankSpanSetSpine)
}
```

# mheap 

还是上面的例子，假如申请 `16b` 内存时，依次经过 `mcache` 和 `mcentral` 都没有可用适宜规则的大小内存，这时候会向 `mheap` 申请一块内存。

然后按指定规格划分为一些列表，并将其添加到相同规格大小的 `mcentral` 的 `not empty list` 后面![](https://blogstatic.haohtml.com/uploads/2021/04/d2b5ca33bd970f64a6301fa75ae2eb22-3.png)mheap

同时，Go 没法使用工作线程的本地缓存 mcache 和全局中心缓存 mcentral 上管理超过32KB的内存分配，所以对于那些超过32KB的内存申请，会直接从堆上(_mheap_)上分配对应的数量的内存页(每页大小是8KB)给程序。

![golang-mheap-allocation](https://blogstatic.haohtml.com//uploads/2023/09/image-20231104151421831.png)

```
type mheap struct {
	// lock must only be acquired on the system stack, otherwise a g
	// could self-deadlock if its stack grows with the lock held.
	lock      mutex
	pages     pageAlloc // page allocation data structure
	sweepgen  uint32    // sweep generation, see comment in mspan; written during STW
	sweepdone uint32    // all spans are swept
	sweepers  uint32    // number of active sweepone calls


	allspans []*mspan // all spans out there

	_ uint32 // align uint64 fields on 32-bit for atomics

	// Proportional sweep
	pagesInUse         uint64  // pages of spans in stats mSpanInUse; updated atomically
	pagesSwept         uint64  // pages swept this cycle; updated atomically
	pagesSweptBasis    uint64  // pagesSwept to use as the origin of the sweep ratio; updated atomically
	sweepHeapLiveBasis uint64  // value of heap_live to use as the origin of sweep ratio; written with lock, read without
	sweepPagesPerByte  float64 // proportional sweep ratio; written with lock, read without

	scavengeGoal uint64

	// Page reclaimer state
	// This is accessed atomically.
	reclaimIndex uint64

	// This is accessed atomically.
	reclaimCredit uintptr


	arenas [1 << arenaL1Bits]*[1 << arenaL2Bits]*heapArena

	heapArenaAlloc linearAlloc

	arenaHints *arenaHint

	arena linearAlloc

	allArenas []arenaIdx

	sweepArenas []arenaIdx

	markArenas []arenaIdx

	curArena struct {
		base, end uintptr
	}

	_ uint32 // ensure 64-bit alignment of central

	central [numSpanClasses]struct {
		mcentral mcentral
		pad      [cpu.CacheLinePadSize - unsafe.Sizeof(mcentral{})%cpu.CacheLinePadSize]byte
	}

	spanalloc             fixalloc // allocator for span*
	cachealloc            fixalloc // allocator for mcache*
	specialfinalizeralloc fixalloc // allocator for specialfinalizer*
	specialprofilealloc   fixalloc // allocator for specialprofile*
	speciallock           mutex    // lock for special record allocators.
	arenaHintAlloc        fixalloc // allocator for arenaHints

	unused *specialfinalizer // never set, just here to force the specialfinalizer type into DWARF
}

var mheap_ mheap
```

 * `lock` 全局锁，保证并发，所以尽量避免从`mheap`中分配
 * `pages` 页分配的数据结构
 * `sweepgen` 清扫生成
 * `sweepdone` 清扫完成标记
 * `sweepers` 活动清扫调用 sweepone 数
 * `allspans` 所有的 spans 都是通过 `mheap_` 申请，所有申请过的 `mspan` 都会记录在 `allspans`，可以随着堆的增长重新分配和移动。结构体中的 `lock` 就是用来保证并发安全的。
 * `pagesInUse` 统计`mSpanInUse` 中`spans`的页数
 * `pagesSwept` 本轮清扫的页数
 * `pagesSweptBasis` 用作清扫率
 * `sweepHeapLiveBasis` 用作扫描率的heap_live 值
 * `sweepPagesPerByte` 清扫率
 * `scavengeGoal` 保留的堆内存总量（预先设定的），runtime 将试图返还内存给OS
 * `reclaimIndex` 指回收的下一页在allAreans 中的索引。具体来说，它指的是 `arena allArenas[i/pagesPerArena]` 的第（`i%pagesPerArena`）页
 * `reclaimCredit` 多余页面的备用信用。因为页回收器工作在大块中，它可能回收的比请求的要多，释放的任何备用页将转到此信用池
 * `arenas [1 << arenaL1Bits]*[1 << arenaL2Bits]*heapArena` **重要字段！**堆arena 映射。它指向整个可用虚拟地址空间的每个 arena 帧的堆元数据；
 使用arenaIndex将索引计算到此数组中；
  对于没有Go堆支持的地址空间区域，arena映射包含`nil`；
  一般来说，这是一个两级映射，由一个L1级映射和多个L2级映射组成；
  当有大量的的 arena 帧时将节省空间，然而在许多平台(64位),arenaL1Bits 是0，这实际上是一个单级映射。这种情况下arenas[0]永远不会为零。
 * `heapArenaAlloc` 是为分配 `heapArena` 对象而预先保留的空间。仅仅用于32位系统。
 * `arenaHints` 试图添加更多堆 arenas 的地址列表。它最初由一组通用少许地址填充，并随实 `heap arena` 的界限而增长。
 * `arena` linearAlloc
 * `allArenas` `[]arenaIdx` 是每个映射arena的arenaIndex 索引。可以用以遍历地址空间。
 * `sweepArenas []arenaIdx` 指在`清扫周期`开始时保留的 `allArenas` 快照
 * `markArenas []arenaIdx` 指在`标记周期`开始时保留的 `allArenas` 快照
 * `curArena` 指heap当前增长时的 `arena`，它总是与`physPageSize`对齐。
 * `central` **重要字段！**这个就是上面介绍的 `mcentral` ，每种规格大小的块对应一个 `mcentral`。pad 是一个字节填充，用来避免伪共享（false sharing）
 * `spanalloc` 数据类型 `fixalloc` 是 free-list，用来分配特定大小的块。比如 `cachealloc` 分配 `mcache` 大小的块。
 * `cachealloc` 同上
 * 其它

对于 `mheap.arenas` 字段对应 `heapArena`类型, 用来存储 `heap arena` 元数据，存储在Go堆的外部，并通过 `mheap.arenas` 索引进行访问。

```
// A heapArena stores metadata for a heap arena. heapArenas are stored
// outside of the Go heap and accessed via the mheap_.arenas index.
//
//go:notinheap
type heapArena struct {
	bitmap [heapArenaBitmapBytes]byte
	spans [pagesPerArena]*mspan

	pageInUse [pagesPerArena / 8]uint8
	pageMarks [pagesPerArena / 8]uint8
	pageSpecials [pagesPerArena / 8]uint8
	checkmarks *checkmarksMap

	zeroedBase uintptr
}
```

 * `heapArena.bitmap` 中每两个 `bit` 对应标记 `arena` 中一个指针大小的`word`（也就是说 `bitmap` 中一个 `byte` 即 8 个位可以标记 `arena` 中连续四个指针大小的内存）；
 每个`word`对应的两个 `bit` 中，低位bit用于标记是否为指针，0为非指针，1为指针；
  高位bit用于标记是否要继续扫描，高位bit为1就代表扫描完当前word并不能完成当前数据对象的扫描；
  ![](https://blogstatic.haohtml.com/uploads/2021/04/d2b5ca33bd970f64a6301fa75ae2eb22-6.png)HeapArena.bitmap

 * `heapArena.spans` 是一个`*mspan` 类型的数组，用于记录当前arena中每一页对应到哪一个`mspan`。
 ![](https://blogstatic.haohtml.com/uploads/2021/04/d2b5ca33bd970f64a6301fa75ae2eb22-7.png)HeapArena.spans

 * `heapArena.pageInUse` 位图类型，指哪些spans 处于 mSpanInUse 状态;
 * `heapArena.pageMarks` 位图类型，指哪些 spans 已被标记。与 `pageInUse` 类似，但只使用每个span中的首页的位;
 * `heapArena.pageSpecials` 位图类型，指哪些spans有specials (finalizers or other).与 `pageInUse` 类似，但只使用每个`span`中的首页的位;
 * `heapArena.checkmarks` 调试使用，仅在 `debug.gccheckmark > 0` 时使用。检查存储 `debug.gccheckmark` 状态；
 * `heapArena.zeroedBase` 标记在arena 首页的第一个字节，未使用

基于 `HeapArena` 记录的元数据信息，我们只要知道一个对象的地址，就可以根据 `HeapArena.bitmap` 信息扫描它内部是否含有指针；也可以根据对象地址计算出它在哪一页，然后通过 `HeapArena.spans` 信息查到该对象存在哪一个`mspan`中。

对于heap结构中的字段比较多，有几个使用频率非常高的字段，如 `allspans`、`arenas`、`allArenas`、`sweepArenas`、`markArenas` 和 `central` 。有些是与GC 有关，有些是与内存维护管理有关。随着阅读 `runtime` 时间的增加，会越来越了解每个字段的使用场景。

# 参考资料 

 * [https://studygolang.com/articles/29752](https://studygolang.com/articles/29752)
 * [https://zhuanlan.zhihu.com/p/338203758](https://zhuanlan.zhihu.com/p/338203758)
 * [https://www.cnblogs.com/shijingxiang/articles/12196677.html](https://www.cnblogs.com/shijingxiang/articles/12196677.html)
 * [http://www.voidcn.com/article/p-yhcodasw-bkx.html](http://www.voidcn.com/article/p-yhcodasw-bkx.html)
 * [https://www.cnblogs.com/zpcoding/p/13259943.html#_label1_4](https://www.cnblogs.com/zpcoding/p/13259943.html#_label1_4)
 * [https://www.dazhuanlan.com/2019/09/29/5d900b0173983/](https://www.dazhuanlan.com/2019/09/29/5d900b0173983/)
 * [https://www.luozhiyun.com/archives/434](https://www.luozhiyun.com/archives/434)