---
title: Golang中的goroutine泄漏问题
author: admin
type: post
date: 2019-11-08T15:00:12+00:00
url: /archives/19308
categories:
 - 程序开发
tags:
 - golang
 - goroutine

---
goroutine作为Go中开发语言中的一大利器，在高并发中发挥着无法忽略的作用。但东西虽好，真正做到用好还是有一些要注意的地方，特别是对于刚刚接触这门开发语言的新手来说，稍有不慎，就极有可能导致goroutine 泄漏。

## 什么是goroutine Leak 

goroutine leak 的意思是go协程泄漏，那么什么又是协程泄漏呢？我们知道每次使用go关键字开启一个gorountine任务，经过一段时间的运行，最终是会结束，从而进行系统资源的释放回收。而如果由于操作不当导致一些goroutine一直处于阻塞状态或者永远运行中，永远也不会结束，这就必定会一直占用系统资源。最球的情况下是随着系统运行，一直在创建此类goroutine，那么最终结果就是程序崩溃或者系统崩溃。这种情况我们一般称为goroutine leak。

## 出现的问题 

先看一段代码：

```
package main

import (
    "fmt"
    "math/rand"
    "runtime"
    "time"
)

func query() int {
    n := rand.Intn(100)
    time.Sleep(time.Duration(n) * time.Millisecond)
    return n
}

// 每次执行此函数，都会导致有两个goroutine处于阻塞状态
func queryAll() int {
    ch := make(chan int)
    go func() { ch <- query() }()
    go func() { ch <- query() }()
    go func() { ch <- query() }()
	// <-ch
	// <-ch
    return <-ch
}

func main() {
    // 每次循环都会泄漏两个goroutine
    for i := 0; i < 4; i++ {
        queryAll()
        // main()也是一个主groutine
        fmt.Printf("#goroutines: %d\n", runtime.NumGoroutine())
    }
}
```

运行结果

```
#goroutines: 3
#goroutines: 5
#goroutines: 7
#goroutines: 9
```

这里发现goroutine的数量一直在增涨，按理说这里的值应该一直是 1 才对的呀(只有一个Main 函数的主goroutine)。其实这里发生了goroutine泄漏的问题。

主要问题发生在 queryAll() 函数里，这个函数在goroutine里往ch里连续三次写入了值，由于这里是无缓冲的ch，所以在写入值的时候，要有在ch有接收者时才可以写入成功，也就是说在从接收者从ch中获取值之前, 前面三个ch<-query() 一直处于阻塞的状态。当执行到queryAll()函数的 return语句 时，ch接收者获取一个值(意思是说三个ch<-query() 中执行最快的那个goroutine写值到ch成功了，还剩下两个执行慢的 ch<-query() 处于阻塞)并返回给调用主函数时，仍有两个ch处于浪费的状态。

**在Main函数中对于for循环**
第一次：goroutine的总数量为 1个主goroutine + 2个浪费的goroutine = 3
第二次：3 + 再个浪费的2个goroutine = 5
第三次：5 + 再个浪费的2个goroutine = 7
第三次：7 + 再个浪费的2个goroutine = 9

正好是程序的输出结果。

好了，问题我们知道怎么回事了，剩下的就是怎么解决了

## 解决方案： 

可以看到，主要是ch写入值次数与读取的值的次数不一致导致的有ch一直处于阻塞浪费的状态，我们所以我们只要保存写与读的次数完全一样就可以了。

这里我们把上面queryAll() 函数代码注释掉的 <-ch 两行取消掉，再执行就正常了，输出内容如下：

```
#goroutines: 1
#goroutines: 1
#goroutines: 1
#goroutines: 1
```

对于goroutine的数量只有一个，也必须有一个，因为Main()函数也是一个goroutine。（）

当然对于解决goroutine的方法不是仅仅这一种，也可以利用context来解决，参考：。

## 使用 uber-go/goleak 工具检测goleak 

上面我们是手动通过获取 groutine数量来判断是否存在泄漏的，下面我们使用 uber-go/goleak工具来检测是否存在泄漏问题

```
func TestGoleak(t *testing.T) {
        // 重要的一条检测goroutine泄漏语句
	defer goleak.VerifyNone(t)

	// chan 长度为0
	ch := make(chan int)
	go func() {
		for i := 0; i < 4; i++ {
			ch <- i
		}
	}()
	go func() {
		for i := 0; i < 3; i++ {
			<-ch
		}
	}()

	time.Sleep(time.Second * 2)

	// 会打印2，因为这里为阻塞chan, 第2个go func(){}只消费了三个，所以当第一个go func(){} 生产第4个数据时会一直牌阻塞状态
	// 此时会有一个goroutine一直处于阻塞运行状态，再加上一个Main()函数的主goroutine,正好是一1个
	fmt.Printf("#goroutines: %d\n", runtime.NumGoroutine())
}
```

工具使用方法只需要在单元测试方法里添加一条“**defer goleak.VerifyNone(t)**” 语句即可。

```
go test -run TestGoleak
#goroutines: 3
--- FAIL: TestGoleak (2.46s)
    leaks.go:78: found unexpected goroutines:
        [Goroutine 8 in state chan send, with iot/pkg.TestGoleak.func1 on top of the stack:
        goroutine 8 [chan send]:
        iot/pkg.TestGoleak.func1(0xc000022300)
                /Users/sxf/iot-server/pkg/a_test.go:39 +0x43
        created by iot/pkg.TestGoleak
                /Users/sxf/iot-server/pkg/a_test.go:37 +0xcd
        ]
FAIL
exit status 1
FAIL    iot/pkg 3.342s
```

从输出内容“**ound unexpected goroutines**”可以看出存在泄漏的goroutine。

如果想一次性执行多个单元测试的话，则需要另外一种方法，就是定义一个 **TestMain()** 函数，如

```
func TestMain(m *testing.M) {
	goleak.VerifyTestMain(m)
}
```

使用此方法的话，就不需要在每个单元测试方法里写 “defer goleak.VerifyNone(t)
”了。TestMain()函数会打印出来每个单元测试方法的检查结果。

上面我们使用了阻塞chan方法，如果你修改为非阻塞，长度为1的话，就不会发生goroutine泄漏的问题了，

另外也可以使用golang/gops 工具，参考,更多用法可以通过 gops –help 命令查看

**提醒：**

垃圾收集器不会收集以下形式的goroutines：

```
go func() {
// <操作会在这里永久阻塞>
}()
// Do work

```

这个goroutine将一直存在，直到整个程序退出，属于一种高级特性。是否属于goroutine leak 还需要看如何使用了，如。如果处理不好，如for{} 根本就不可能结束，就算泄漏，所以我们写程序时，至少要保证他们结束的条件，且一定可以结束才算正常。

## 产生goroutine leak的原因 

 * goroutine由于channel的读/写端退出而一直阻塞，导致goroutine一直占用资源，而无法退出，如只有写入，没有接收，反之一样
 * goroutine进入死循环中，导致资源一直无法释放

## goroutine终止的场景 

 * goroutine完成它的工作
 * 由于发生了没有处理的错误
 * 收到结束信号，直接终止任务

## 检测工具

如何定义Golang中的goroutine中的泄漏概念？ 有没有相应的泄漏检测工具？可以参考 https://blog.51cto.com/13778063/2152654、 https://github.com/google/gops  或https://github.com/uber-go/goleak

**参考**
* https://repl.it/@pllv/Google-Search-Gorountine-Parallel-Replicas-Rob-Pike
* https://studygolang.com/articles/12495
* https://blog.csdn.net/qq_16205285/article/details/90113419
* https://segmentfault.com/q/1010000007735676
* https://www.cnblogs.com/chenqionghe/p/9769351.html
* https://www.kancloud.cn/mutouzhang/go/596824
