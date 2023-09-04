---
title: Golang环境变量之GODEBUG
author: admin
type: post
date: 2021-01-26T09:15:11+00:00
url: /archives/21778
categories:
 - 程序开发
tags:
 - godebug
 - golang

---
`GODEBUG` 是 golang中一个控制runtime调度变量的变量，其值为一个用逗号隔开的 name=val对列表，常见有以下几个命名变量。

## allocfreetrace 

设置`allocfreetrace = 1`会导致对每个分配进行概要分析，并在每个对象的分配上打印堆栈跟踪并释放它们。

## clobberfree 

设置 `clobberfree=1`会使垃圾回收器在释放对象的时候，对象里的内存内容可能是错误的。

## cgocheck 

cgo相关。

设置 `cgocheck=0` 将禁用当包使用cgo非法传递给go指针到非go代码的检查。如果值为1(默认值)会启用检测，但可能会丢失有一些错误。如果设置为2的话，则不会丢失错误。但会使程序变慢。

## efence 

设置 `efence=1`会使回收器运行在一个模式。每个对象都在一个唯一的页和地址，且永远也不会被回收。

## gccheckmark 

GC相关。

设置 `gccheckmark=1` 启用验证垃圾回收器的并发标记，通过在STW时第二个标记阶段来实现，如果在第二阶段的时候，找到一个可达对象，但未找到并发标记，则GC会发生Panic。

## gcpacertrace 

GC相关。

设置 `gcpacertrace=1`可使gc打印关于并发 pacer 内部状态的信息

## gcshrinkstackoff 

GC相关。

设置 `gcshrinkstackoff=1可`禁止goroutine移动到小stack。这种模式下，goroutine stack只能增长。简单来说就是一个关闭收缩的开关。

## gcstoptheworld 

GC 相关。

设置 `gcstoptheworld=1`可禁止并发垃圾回收，使每个gc就是一个事件。设置 `gcstoptheworld=2` 会在垃圾回收后进行并发扫描。

## gctrace 

GC 相关。

设置 `gctrade=1` 会使垃圾回收器在每次GC打印单行标准错误，汇总收集的内存量和暂停的时间。格式可能会更换。

如果输出行以(`forced`)结尾，则说明此GC是通过调用 runtime.GC 来触发的。这个调试变量非常常见。如 `GODEBUG=gotrace=1 go run main.go`

```
输出字段意义:
	gc # @#s #%: #+#+# ms clock, #+#/#/#+# ms cpu, #->#-># MB, # MB goal, # P
where the fields are as follows:
	gc #        the GC number, incremented at each GC
	@#s         time in seconds since program start
	#%          percentage of time spent in GC since program start
	#+...+#     wall-clock/CPU times for the phases of the GC
	#->#-># MB  heap size at GC start, at GC end, and live heap
	# MB goal   goal heap size
	# P         number of processors used
The phases are stop-the-world (STW) sweep termination, concurrent
mark and scan, and STW mark termination. The CPU times
for mark/scan are broken down in to assist time (GC performed in
line with allocation), background GC time, and idle GC time.
If the line ends with "(forced)", this GC was forced by a
runtime.GC() call.
```

```
➜  gotest GODEBUG=gctrace=1 go run main.go
gc 1 @0.115s 0%: 0.067+0.92+0.003 ms clock, 0.26+0.41/0.78/0.011+0.015 ms cpu, 4->4->0 MB, 5 MB goal, 4 P
gc 2 @0.194s 0%: 0.056+1.5+0.003 ms clock, 0.22+1.3/0.27/1.6+0.012 ms cpu, 4->4->0 MB, 5 MB goal, 4 P
gc 3 @0.303s 0%: 0.24+1.5+0.004 ms clock, 0.98+0.20/0.32/1.5+0.016 ms cpu, 4->4->0 MB, 5 MB goal, 4 P
gc 4 @0.344s 0%: 0.12+0.45+0.003 ms clock, 0.51+0/0.41/0.81+0.012 ms cpu, 4->4->0 MB, 5 MB goal, 4 P
```

字段解释

垃圾回收信息
`gc 1 @0.115s 0%: 0.018+1.3+0.076 ms clock, 0.054+0.35/1.0/3.0+0.23 ms cpu, 4->4->3 MB, 5 MB goal, 4 P`。

`1` ：表示第一次执行
`@0.115s` ：表示当前程序执行的总时间，从程序开始时计时。
`0%` ：垃圾回收时间占用的百分比
`0.067+0.92+0.003 ms clock` ：垃圾回收的时间，分别为STW（stop-the-world）清扫时间, 并发标记和扫描的时间，STW标记的时间
`0.26+0.41/0.78/0.011+0.015 ms cpu` ：垃圾回收占用cpu时间
`4->4->0 MB` ：堆的大小，三个数字分别表示GC开始前堆的大小，GC后堆的大小 和 当前存活堆的大小
`5 MB goal` ：整体堆的大小
`4 P` ：使用的处理器数量

在gc后的内存回收到系统时，会有一些概要信息，如
`scvg0: inuse: 426, idle: 0, sys: 427, released: 0, consumed: 427 (MB)`

`426` ：使用多少M内存
`` ：剩下要清除的内存
`427` ：系统映射的内存
`` ：释放的系统内存
`427` ：申请的系统内存

## madvdontneed 

设置 `madvdontneed=1` 将会在Linux上当内存返回内核时使用 `MADV_DONTNEED` 代替 `MADV_FREE`。

## memprofilerate 

设置`memprofilerate=X` 将会更新 `runtime.MemProfileRate` 的值。如果设置为 `` 将禁用内存性能分析。有关默认值，请参考 `MemProfileRate` 的描述。

## invalidptr 

// TODO

## sbrk 

设置`sbrk=1`会使用一个`碎片回收器`代替内存分配器和垃圾回收器。它从操作系统获取内存，并且永远也不会回收任何内存。

## scavenge 

设置 `scavenge=1` 将启用`heap scavenger`的调试模式。

## scavtrace 

//TODO

## scheddetail 

调度相关。

设置为`schedtrace=X` 和 `scheddetail=1` 会使调度器每 X 毫秒打印详细的多行信息，调度器的描述状态，处理器、线程和goroutines。

```
package main

import "sync"

func main() {
	wg := sync.WaitGroup{}
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func(wg *sync.WaitGroup) {
			var counter int
			for i := 0; i < 1e10; i++ {
				counter++
			}
			wg.Done()
		}(&wg)
	}

	wg.Wait()
}
```

输出结果

```
➜  gotest GODEBUG=schedtrace=1000,scheddetail=1000 go run main.go
SCHED 0ms: gomaxprocs=4 idleprocs=2 threads=6 spinningthreads=1 idlethreads=2 runqueue=0 gcwaiting=0 nmidlelocked=0 stopwait=0 sysmonwait=0
  P0: status=1 schedtick=1 syscalltick=0 m=0 runqsize=0 gfreecnt=0 timerslen=0
  P1: status=0 schedtick=2 syscalltick=0 m=-1 runqsize=0 gfreecnt=0 timerslen=0
  P2: status=1 schedtick=0 syscalltick=0 m=5 runqsize=0 gfreecnt=0 timerslen=0
  P3: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=0 gfreecnt=0 timerslen=0
  M5: p=2 curg=-1 mallocing=0 throwing=0 preemptoff= locks=1 dying=0 spinning=true blocked=false lockedg=-1
  M4: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=true lockedg=-1
  M3: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=true lockedg=-1
  M2: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=2 dying=0 spinning=false blocked=false lockedg=-1
  M1: p=-1 curg=17 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=false lockedg=17
  M0: p=0 curg=1 mallocing=0 throwing=0 preemptoff= locks=3 dying=0 spinning=false blocked=false lockedg=1
  G1: status=2(chan receive) m=0 lockedm=0
  G17: status=6() m=1 lockedm=1
  G2: status=4(force gc (idle)) m=-1 lockedm=-1
  G3: status=4(GC sweep wait) m=-1 lockedm=-1
  G4: status=4(GC scavenge wait) m=-1 lockedm=-1
SCHED 0ms: gomaxprocs=4 idleprocs=1 threads=5 spinningthreads=1 idlethreads=0 runqueue=0 gcwaiting=0 nmidlelocked=0 stopwait=0 sysmonwait=0
  P0: status=1 schedtick=0 syscalltick=0 m=4 runqsize=1 gfreecnt=0 timerslen=0
  P1: status=1 schedtick=0 syscalltick=0 m=2 runqsize=0 gfreecnt=0 timerslen=0
  P2: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=0 gfreecnt=0 timerslen=0
  P3: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=0 gfreecnt=0 timerslen=0
  M4: p=0 curg=-1 mallocing=0 throwing=0 preemptoff= locks=1 dying=0 spinning=false blocked=false lockedg=-1
```

输出的内容比较的多，每行的信息属于谁只要看首列字母即。如果是`G`开头的话，则是表示的是 goroutine, G0、G1、GN后面的数字表示是G的编号，是唯一的；如果是`P`的话，则就是处理器相关的信息、如果是 `M` 则表示线程了，每种资源都有多个，他们之间状态是来回变化的，这是由于阻塞或调度的原因。SCHED 则表示当前的调度信息。

建议看一下每行相关的字段意义，这样有助于理解 runtime。如 P 行输出行里的字段 `status`、`schedtick`、`syscalltick`、`m` 和 `runqsize` 全是 P 结构体的字段。

**P相关字段**

 * `status` 当前P的状态，参考 [这里](https://blog.haohtml.com/archives/21010#P)
 * `schedtick` 调度次数
 * `syscalltick` P系统调用次数
 * `m` 当前P绑定的M
 * `runqsize` P本地的运行队列，存放的全是待执行的Goroutine
 * `gfreecnt` 可用的G（状态Gdead）

**M相关字段**

 * `p` 当前m绑定的P
 * `curg` 当前绑定的G
 * `mallocing` 分配内存状态
 * `throwing` 是否抛出异常
 * `preemptoff` 如果非空，则保持curg 运行在当前m。
 * `locks` 锁
 * `dying` 死亡?
 * `spinning` 是否自旋
 * `blocked` 当前阻塞
 * `lockedg` 当前锁定的g


**G相关字段**

 * `status` 当前G 的状态，参考 [这里](https://blog.haohtml.com/archives/21010#G)
 * `m` 当前G分配到的M, -1表示未分配
 * `lockedm` 当前G锁定的m

## schedtrace 

调度相关。

设置 `schedtrade=X` 会使调度器每X毫秒打印一行标准错误，汇总调度器的状态。此参数的打印值则函数 ` [schedtrace()](https://github.com/golang/go/blob/go1.16.2/src/runtime/proc.go#L5389-L5468) ` 负责。

```
package main

import "sync"

func main() {
	wg := sync.WaitGroup{}
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func(wg *sync.WaitGroup) {
			var counter int
			for i := 0; i < 1e10; i++ {
				counter++
			}
			wg.Done()
		}(&wg)
	}

	wg.Wait()
}

```

输出结果

```
➜  gotest GODEBUG=schedtrace=1000 go run main.go
SCHED 0ms: gomaxprocs=4 idleprocs=2 threads=4 spinningthreads=1 idlethreads=0 runqueue=0 [1 0 0 0]
SCHED 1ms: gomaxprocs=4 idleprocs=2 threads=7 spinningthreads=1 idlethreads=2 runqueue=0 [0 0 0 0]
SCHED 3ms: gomaxprocs=4 idleprocs=2 threads=7 spinningthreads=1 idlethreads=2 runqueue=0 [0 0 0 0]
SCHED 10ms: gomaxprocs=4 idleprocs=1 threads=7 spinningthreads=1 idlethreads=1 runqueue=0 [7 7 0 0]
SCHED 15ms: gomaxprocs=4 idleprocs=0 threads=7 spinningthreads=1 idlethreads=0 runqueue=0 [3 1 3 0]
SCHED 18ms: gomaxprocs=4 idleprocs=0 threads=7 spinningthreads=0 idlethreads=0 runqueue=0 [1 1 3 1]
SCHED 22ms: gomaxprocs=4 idleprocs=0 threads=8 spinningthreads=0 idlethreads=0 runqueue=1 [1 1 3 1]
SCHED 24ms: gomaxprocs=4 idleprocs=0 threads=8 spinningthreads=0 idlethreads=1 runqueue=0 [0 0 0 0]
SCHED 25ms: gomaxprocs=4 idleprocs=0 threads=8 spinningthreads=0 idlethreads=1 runqueue=0 [0 0 0 0]
SCHED 29ms: gomaxprocs=4 idleprocs=0 threads=8 spinningthreads=1 idlethreads=0 runqueue=0 [0 6 0 0]
SCHED 30ms: gomaxprocs=4 idleprocs=1 threads=8 spinningthreads=1 idlethreads=1 runqueue=0 [0 0 0 0]
......
```

我们这时设置的是 schedtrace=1000,即1秒输出一次，

 * `SCHED`：每一行都表示一次调度器的调试信息，后面提示的毫秒数表示启动到现在的运行时间，输出的时间间隔受 `schedtrace` 的值影响。
 * `gomaxprocs`：当前的 CPU 核心数（GOMAXPROCS 的当前值）。
 * `idleprocs`：空闲的处理器数量，后面的数字表示当前的空闲数量。
 * `threads：OS` 线程数量，后面的数字表示当前正在运行的线程数量。
 * `spinningthreads`：自旋状态的 OS 线程数量。
 * `idlethreads`：空闲的线程数量。
 * `runqueue`：全局队列中 Goroutine 数量，而后面的 [3 1 3 0] 则分别代表这 4 个 P 的本地队列正在运行的 Goroutine 数量，其值由 p.runqtail – p.runqhead 计算而得。

如果想打印更详情的信息，可能可以与 `scheddetail` 一起使用，如`GODEBUG=schedtrace=1000,scheddetail=1 go run main.go` ，会打印出每个G/P/M之间的切换状态详情。

推荐使用pprof查看，目前这种方式不是太友好。

## tracebackancestors 

//TODO

## asyncpreemptoff 

异步抢占

以上几个调试变量，经常使用的有`gotrace`、`schedtrace` 和 `scheddetail`，其中gotrace主要是观察gc的情况，schedtrace 和 scheddetail 主要是观察调度的情况。

另外`net`、`net/http`和 `crypto/tls` 包也引用了GODEUBG中的调度变量，具体参考包的文档。

## 参考 

 * https://pkg.go.dev/runtime@go1.15.6
 * [https://blog.csdn.net/EDDYCJY/article/details/102426078](https://blog.csdn.net/EDDYCJY/article/details/102426078)