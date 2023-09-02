---
title: Golang中的CAS原子操作 和 锁
author: admin
type: post
date: 2021-03-28T10:58:34+00:00
url: /archives/25881
categories:
 - 程序开发
tags:
 - golang
 - mutex

---
在高并发编程中，经常会出现对同一个资源并发访问修改的情况，为了保证最终结果的正确性，一般会使用 `锁` 和 `CAS原子操作` 来实现。

如要对一个变量进行计数统计，两种实现方式分别为

```
package main

import (
	"fmt"
	"sync"
)

// 锁实现方式
func main() {
	var count int64
	var wg sync.WaitGroup
	var mu sync.Mutex

	for i := 0; i < 10000; i++ {
		wg.Add(1)
		go func(wg *sync.WaitGroup) {
			defer wg.Done()
			mu.Lock()
			count = count + 1
			mu.Unlock()
		}(&wg)
	}
	wg.Wait()

	// count = 10000
	fmt.Println("count = ", count)
}

```

与

```
package main

import (
	"fmt"
	"sync"

	"sync/atomic"
)

// atomic CAS 原子操作
func main() {
	var count int64
	var wg sync.WaitGroup

	for i := 0; i < 10000; i++ {
		wg.Add(1)
		go func(wg *sync.WaitGroup) {
			defer wg.Done()
			// 失败一直重试
			for {
				old := atomic.LoadInt64(&count)
				if atomic.CompareAndSwapInt64(&count, old, old+1) {
					break
				}
			}

		}(&wg)
	}
	wg.Wait()

	// count = 10000
	fmt.Println("count = ", count)
}

```

可以看到两种用法的执行结果是一样的，我们再看一下两者的性能区别。

# 性能差别 

```
package main_test

import (
	"sync"
	"sync/atomic"
	"testing"
)

func lockTest() {
	var count int64
	var wg sync.WaitGroup
	var mu sync.Mutex

	for i := 0; i < 10000; i++ {
		wg.Add(1)
		go func(wg *sync.WaitGroup) {
			defer wg.Done()
			mu.Lock()
			count = count + 1
			mu.Unlock()
		}(&wg)
	}
	wg.Wait()
}
func atomicTest() {
	var count int64
	var wg sync.WaitGroup

	for i := 0; i < 10000; i++ {
		wg.Add(1)
		go func(wg *sync.WaitGroup) {
			defer wg.Done()
			// 通过
			for {
				old := atomic.LoadInt64(&count)
				if atomic.CompareAndSwapInt64(&count, old, old+1) {
					break
				}
			}

		}(&wg)
	}
	wg.Wait()
}

func BenchmarkLock(b *testing.B) {
	for i := 0; i < b.N; i++ {
		lockTest()
	}

}
func BenchmarkAtomic(b *testing.B) {
	for i := 0; i < b.N; i++ {
		atomicTest()
	}
}

```

下面分别用一个、两个、四个CPU来测试他们两者的性能差别

## 一个CPU 

```
➜  gotest go test -bench=".*" -v -benchmem -cpu=1
goos: darwin
goarch: amd64
pkg: gotest
cpu: Intel(R) Core(TM) i5-5257U CPU @ 2.70GHz
BenchmarkLock
BenchmarkLock                194           6312940 ns/op              32 B/op          3 allocs/op
BenchmarkAtomic
BenchmarkAtomic              189           6342975 ns/op              24 B/op          2 allocs/op
PASS
ok      gotest  3.296s
```

在只用一个CPU的情况下，看着差别不是太大，每次 op 的内存使用 atomic 比 lock 少了1/3。

## 两个CPU 

```
➜  gotest go test -bench=".*" -v -benchmem -cpu=2
goos: darwin
goarch: amd64
pkg: gotest
cpu: Intel(R) Core(TM) i5-5257U CPU @ 2.70GHz
BenchmarkLock
BenchmarkLock-2              350           3178037 ns/op            2071 B/op          7 allocs/op
BenchmarkAtomic
BenchmarkAtomic-2            364           3035188 ns/op             439 B/op          3 allocs/op
PASS
ok      gotest  3.370s
```

可以看到相比一个CPU来说，每个 op 的时间基于快了一倍，分配的内存也大大减小了。

## 四个CPU 

```
➜  gotest go test -bench=".*" -v -benchmem -cpu=4
goos: darwin
goarch: amd64
pkg: gotest
cpu: Intel(R) Core(TM) i5-5257U CPU @ 2.70GHz
BenchmarkLock
BenchmarkLock-4              349           3462298 ns/op            3613 B/op         17 allocs/op
BenchmarkAtomic
BenchmarkAtomic-4            330           3382266 ns/op              24 B/op          2 allocs/op
PASS
ok      gotest  4.221s
```

使用四个CPU相比使用两个，`op` 基本没有太大的变化，但 `op` 和 `allocs` 两个却十分明显。

## 总结 

从结果来看当使用多个CPU的时候，差距最为明显的是分配内存的次数和内存的大小，总体来看使用 `atomic` 要比使用 `lock` 的性能要好。

所以在只修改一个变量值的单个指令场景下，推荐使用 `atomic` ，而不是 `lock` 锁。

# 实现原理 

原子操作一般是由 `硬件底层` 支持的，而锁则是由操作系统层面来实现的。比起使用锁，使用 CAS原子操作这个过程是不会形成临界区和创建临界区的，大大减少了同步对程序性能的影响，所以性能要高效一些。

但原子也有一定的弊端，在被操作值频繁变更的情况下，很可能失败，需要一直重试直到成功为止，这种重试行为也可以理解为自旋spinning，长时间处于spinning将浪费CPU，参考 [https://www.bilibili.com/video/BV1Sf4y1s7Np/](https://www.bilibili.com/video/BV1Sf4y1s7Np/)。

## 原子操作 

从硬件的层面实现原子操作，有两种方式：

 1. 总线加锁：因为CPU和其他硬件的通信都是通过总线控制的，所以可以通过在总线加 `LOCK#` 锁的方式实现原子操作，但这样会阻塞其他硬件对CPU的访问，开销比较大。
 2. 缓存锁定：频繁使用的内存会被处理器放进高速缓存中，那么原子操作就可以直接在处理器的高速缓存中进行而不需要使用总线锁，主要依靠缓存一致性来保证其原子性。（ [MESI协议](https://www.scss.tcd.ie/Jeremy.Jones/VivioJS/caches/MESIHelp.htm)）

现在几乎所有的CPU指令都支持CAS的原子操作，`X86`下对应的是 `CMPXCHG` 汇编指令。有了这个原子操作，我们就可以用其来实现各种 [无锁（lock free）](https://blog.haohtml.com/archives/30670) 的数据结构。

CAS 在Golang中是以共享内存的方式来实现的一种同步机制，它是一个原子操作，一般格式如下

```
fun addValue(delta int32){
    for{
        oldValue := atomic.LoadInt32(&addr)
        if atomic.CompareAndSwapInt32(&addr, oldValue, oldValue+delta){
            break;
        }
    }
}
```

先从一个内存地址 `&addr` 读取出来当前存储的值，假如读取完以后，没有其它线程对此变量 进行修改的话，则下面的 `atomic.CompareAndSwapInt32` 语句会在执行时先再判断一次它的值是否与上次的相等，这时必须是相等的，则直接更新它的值；如果在读取值后，有其它线程对变量值进行了修改，发现值不相等，这时就再重新开始下一轮的判断，直到修改成功为止。

对于 `atomic.CompareAndSwapIntxx()` 之类函数的实现是在 ` [src/sync/atomic.asm.s](https://github.com/golang/go/blob/go1.15.6/src/sync/atomic/asm.s#L24-L37) ` 文件里声明的，真正对应的汇编文件位于 ` [src/runtime/internal/atomic/*.s](https://github.com/golang/go/blob/go1.15.6/src/runtime/internal/atomic) `，如64架构对应的文件为 ` [asm_amd64.s](https://github.com/golang/go/blob/go1.15.6/src/runtime/internal/atomic/asm_amd64.s) `。

`atomic.CompareAndSwapInt32` 和 `atomic.CompareAndSwapUint32` 对应的汇编是 `runtime∕internal∕atomic·Cas`;

```
// bool Cas(int32 *val, int32 old, int32 new)
// Atomically:
//	if(*val == old){
//		*val = new;
//		return 1;
//	} else
//		return 0;
TEXT runtime∕internal∕atomic·Cas(SB),NOSPLIT,$0-17
	MOVQ	ptr+0(FP), BX
	MOVL	old+8(FP), AX
	MOVL	new+12(FP), CX
	LOCK
	CMPXCHGL	CX, 0(BX)
	SETEQ	ret+16(FP)
	RET
```

`atomic.CompareAndSwapInt64` 和 `atomic.CompareAndSwapUint64` 对应的汇编是 `runtime∕internal∕atomic·Cas64`；

```
// bool	runtime∕internal∕atomic·Cas64(uint64 *val, uint64 old, uint64 new)
// Atomically:
//	if(*val == *old){
//		*val = new;
//		return 1;
//	} else {
//		return 0;
//	}
TEXT runtime∕internal∕atomic·Cas64(SB), NOSPLIT, $0-25
	MOVQ	ptr+0(FP), BX
	MOVQ	old+8(FP), AX
	MOVQ	new+16(FP), CX
	LOCK
	CMPXCHGQ	CX, 0(BX)
	SETEQ	ret+24(FP)
	RET

```

可以看到汇编相关的指令有 `LOCK` 和 `CMPXCHGQ` 及其它几个指令。

这里的LOCK 称为 LOCK指令前缀，是用来设置CPU的 `LOCK#` 信号的（ [见这里](https://blog.csdn.net/imred/article/details/51994189)）。（译注：这个信号会使总线锁定，阻止其他处理器接管总线访问内存），直到使用LOCK前缀的指令执行结束，这会使这条指令的执行变为原子操作。在多处理器环境下，设置 `LOCK#` 信号能保证某个处理器对共享内存的独占使用。

`CMPXCHG` 是比较并交换指令， [参考这里](https://blog.csdn.net/u011624903/article/details/113321235)。

## 锁 

上面我们提到过锁的实现是由操作系统来实现的，所以它的锁粒度要保证最小越好。对锁的理解很简单，这里就不再详细介绍了。

# 参考资料 

 * [https://zhuanlan.zhihu.com/p/94976168](https://zhuanlan.zhihu.com/p/94976168)
 * [https://www.bilibili.com/video/BV1ef4y147vh](https://www.bilibili.com/video/BV1ef4y147vh)
 * [https://www.scss.tcd.ie/Jeremy.Jones/VivioJS/caches/MESIHelp.htm](https://www.scss.tcd.ie/Jeremy.Jones/VivioJS/caches/MESIHelp.htm)
 * [https://tech.meituan.com/2018/11/15/java-lock.html](https://tech.meituan.com/2018/11/15/java-lock.html)
 * [https://segmentfault.com/a/1190000021653471](https://segmentfault.com/a/1190000021653471)
 * [https://blog.csdn.net/imred/article/details/51994189](https://blog.csdn.net/imred/article/details/51994189)