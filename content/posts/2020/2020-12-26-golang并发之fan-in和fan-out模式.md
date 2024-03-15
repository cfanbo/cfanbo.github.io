---
title: Golang并发模式之扇入FAN-IN和扇出FAN-OUT
author: admin
type: post
date: 2020-12-26T10:38:43+00:00
url: /archives/20363
categories:
 - 程序开发
tags:
 - golang

---

在现实世界中，经常有一些工作是属于流水线类型的，它们的每一个步骤都是紧密关联的，第一步先做什么，再做什么，最后做什么。特别是制造业这个行业，基本全是流水线生产车间。在我们开发中也经常遇到这类的业务场景。

假如我们有个流水线共分三个步骤，分别是 job1、job2和job3。代码： [https://play.golang.org/p/e7ZlP9ofXB3](https://play.golang.org/p/e7ZlP9ofXB3)

```go
package main

import (
	"fmt"
	"time"
)

func job1(count int) <-chan int {
	outCh := make(chan int, 2)

	go func() {
		defer close(outCh)
		for i := 0; i < count; i++ {
			time.Sleep(time.Second)
			fmt.Println("job1 finish:", 1)
			outCh <- 1
		}
	}()

	return outCh
}

func job2(inCh <-chan int) <-chan int {
	outCh := make(chan int, 2)

	go func() {
		defer close(outCh)
		for val := range inCh {
			// 耗时2秒
			time.Sleep(time.Second * 2)
			val++
			fmt.Println("job2 finish:", val)
			outCh <- val
		}
	}()

	return outCh
}

func job3(inCh <-chan int) <-chan int {
	outCh := make(chan int, 2)

	go func() {
		defer close(outCh)
		for val := range inCh {
			val++
			fmt.Println("job3 finish:", val)
			outCh <- val
		}
	}()

	return outCh
}

func main() {
	t := time.Now()

	firstResult := job1(10)
	secondResult := job2(firstResult)
	thirdResult := job3(secondResult)

	for v := range thirdResult {
		fmt.Println(v)
	}

	fmt.Println("all finish")
	fmt.Println("duration:", time.Since(t).String())
}
```

输出结果为

```
job1 finish: 1
job1 finish: 1
job1 finish: 1
job2 finish: 2
job3 finish: 3
3
job1 finish: 1
job1 finish: 1
job2 finish: 2
job3 finish: 3
3
job1 finish: 1
job2 finish: 2
job3 finish: 3
3
job1 finish: 1
job2 finish: 2
job3 finish: 3
3
job1 finish: 1
job2 finish: 2
job3 finish: 3
3
job1 finish: 1
job2 finish: 2
job3 finish: 3
3
job1 finish: 1
job2 finish: 2
job3 finish: 3
3
job2 finish: 2
job3 finish: 3
3
job2 finish: 2
job3 finish: 3
3
job2 finish: 2
job3 finish: 3
3
all finish
duration: 21s
```

共计计算 `21秒`。主要是因为job2中的耗时太久导致，现在我们的主要任务就是解决掉这个问题了。
这里只用了一个job2来处理job1的结果，如果我们能多开启几个goroutine job2并行处理会不会提升性能呢?

现在我们改进下代码，解决job2耗时的问题，需要注意一下，这里对ch的关闭也要作一下调整，由于启用了多个job2的goroutine，所以在job2内部进行关闭了。代码 [https://play.golang.org/p/qebQ00v973C](https://play.golang.org/p/qebQ00v973C)

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func job1(count int) <-chan int {
	outCh := make(chan int, 2)

	go func() {
		defer close(outCh)
		for i := 0; i < count; i++ {
			time.Sleep(time.Second)
			fmt.Println("job1 finish:", 1)
			outCh <- 1
		}
	}()

	return outCh
}

func job2(inCh <-chan int) <-chan int {
	outCh := make(chan int, 2)

	go func() {
		defer close(outCh)
		for val := range inCh {
			// 耗时2秒
			time.Sleep(time.Second * 2)
			val++
			fmt.Println("job2 finish:", val)
			outCh <- val
		}
	}()

	return outCh
}

func job3(inCh <-chan int) <-chan int {
	outCh := make(chan int, 2)

	go func() {
		defer close(outCh)
		for val := range inCh {
			val++
			fmt.Println("job3 finish:", val)
			outCh <- val
		}
	}()

	return outCh
}

func merge(inCh ...<-chan int) <-chan int {
	outCh := make(chan int, 2)

	var wg sync.WaitGroup
	for _, ch := range inCh {
		wg.Add(1)
		go func(wg *sync.WaitGroup, in <-chan int) {
			defer wg.Done()
			for val := range in {
				outCh <- val
			}
		}(&wg, ch)
	}

	// 重要注意，wg.Wait() 一定要在goroutine里运行，否则会引起deadlock
	go func() {
		wg.Wait()
		close(outCh)
	}()

	return outCh
}

func main() {
	t := time.Now()

	firstResult := job1(10)

	// 拆分成三个job2，即3个goroutine （扇出），可以理解成为 job2 搞三个车间
	secondResult1 := job2(firstResult)
	secondResult2 := job2(firstResult)
	secondResult3 := job2(firstResult)

	// 合并结果(扇入）
	secondResult := merge(secondResult1, secondResult2, secondResult3)

	thirdResult := job3(secondResult)

	for v := range thirdResult {
		fmt.Println(v)
	}

	fmt.Println("all finish")
	fmt.Println("duration:", time.Since(t).String())
}
```

输出结果

```
job1 finish: 1
job1 finish: 1
job1 finish: 1
job2 finish: 2
job3 finish: 3
3
job1 finish: 1
job2 finish: 2
job3 finish: 3
3
job1 finish: 1
job2 finish: 2
job3 finish: 3
3
job1 finish: 1
job2 finish: 2
job3 finish: 3
3
job1 finish: 1
job2 finish: 2
job3 finish: 3
3
job1 finish: 1
job2 finish: 2
job3 finish: 3
3
job1 finish: 1
job2 finish: 2
job3 finish: 3
3
job1 finish: 1
job2 finish: 2
job3 finish: 3
3
job2 finish: 2
job3 finish: 3
3
job2 finish: 2
job3 finish: 3
3
all finish
duration: 12s
```

可以看到，性能提升了90%，由原来的 `22s` 减少到 `12s`。上面代码中为了演示效果，使用的缓冲chan很小，如果调大的话，性能更明显。通过这种方式大大提高了工作效率。

**FAN-OUT模式：** 多个goroutine从同一个通道读取数据，直到该通道关闭。**OUT** 是一种张开的模式（1对多），所以又被称为扇出，可以用来分发任务。

**FAN-IN模式：** 1个goroutine从多个通道读取数据，直到这些通道关闭。**IN** 是一种收敛的模式（多对1），所以又被称为扇入，用来收集处理的结果。

是不是很像扇子的状态, 先展开(扇出)再合并(扇入)。

总结：在类似流水线这类的逻辑中，我们可以使用FAN-IN和FAN-OUT模式来提升程序性能。

## 参考 

 * [https://go.dev/blog/pipelines](https://go.dev/blog/pipelines)
 * https://coolshell.cn/articles/21228.html#Fan_in/Out