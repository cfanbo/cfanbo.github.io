---
title: golang中几种对goroutine的控制方法
author: admin
type: post
date: 2020-05-27T03:36:16+00:00
url: /archives/20047
categories:
 - 程序开发
tags:
 - golang

---
我们先看一段代码

```
func listen() {
	ticker := time.NewTicker(time.Second)
	for {
		select {
		case <-ticker.C:
			fmt.Println(time.Now())
		}
	}
}
func main() {
	go listen()
	time.Sleep(time.Second * 5)
	fmt.Println("main exit")
}
```

非常简单的一个goroutine用法，想必每个gopher都看过的。

不过在实际生产中，我们几乎看不到这种用法的的身影，原因很简单，我们无法实现对goroutine的控制，而一般业务中我们需要根据不同情况对goroutine进行各种操作。

要实现对goroutine的控制，一般有以下两种。

## 一、手动发送goroutine控制信号 

这里我们发送一个退出goroutine的信号。

```
// listen 利用只读chan控制goroutine的退出
func listen(ch <-chan bool) {
	ticker := time.NewTicker(time.Second)
	for {
		select {
		case <-ticker.C:
			fmt.Println(time.Now())
		case <-ch:
			fmt.Println("goroutine exit")
			return
		}
	}
}

func main() {
	// 声明一个控制goroutine退出的chan
	ch := make(chan bool, 1)
	go listen(ch)

	// 只写chan
	func(ch chan<- bool) {
		time.Sleep(time.Second * 3)
		fmt.Println("发送退出chan信号")
		ch <- true
		close(ch)
	}(ch)

	time.Sleep(time.Second * 5)
	fmt.Println("main exit")
}
```

我们在main函数里发送一个控制goroutine的退出信息，在goroutine里我们会select这个通道，如果收到此信息，则直接退出goroutine。这里我们使用到了单向chan。

## 二、利用 context 包来控制goroutine 

```
func main() {
	ctx, cancel := context.WithTimeout(context.Background(), time.Second*3)
	defer cancel()

	go func() {
		ticker := time.NewTicker(time.Second)
		for {
			select {
			case <-ticker.C:
				fmt.Println(time.Now())
			case <-ctx.Done():
				// 3秒后会收到信号
				fmt.Println("goroutine exit")
				return
			}
		}
	}()

	time.Sleep(time.Second * 5)
	fmt.Println("main exit")
}
```

这里主要用到了上下文包context来实现，如果你看过包源码的话会发现 ctx.Done() 其实也是chan的读取操作，其原理和第一种方法是完全一样。