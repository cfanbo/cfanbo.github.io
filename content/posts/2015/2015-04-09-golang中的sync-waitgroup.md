---
title: golang中的sync.WaitGroup
author: admin
type: post
date: 2015-04-08T18:19:28+00:00
url: /archives/15583
categories:
 - 程序开发

---
Golang的sync的包有一个并发原语WaitGroup，在日常开发中比较的有用。

WaitGroup的用途：它能够一直等到所有的goroutine执行完成，在其期间会会阻塞主线程的执行，直到所有的goroutine执行完成。

**这里要注意一下，在其中的多个goroutine 的执行结果是没有顺序的，调度器不能保证多个 goroutine 执行次序，且进程退出时不会等待它们结束。**

WaitGroup总共有三个方法：`Add(delta int),` `Done()` 和 `Wait()`。简单的说一下这三个方法的作用。

> Add: 添加或者减少等待goroutine的数量
>
>
> Done: 相当于Add(-1)
>
>
> Wait: 执行阻塞，直到所有的WaitGroup数量变成0

**举个例子**

```
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    var wg sync.WaitGroup

    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(n int) {
            defer wg.Done(),注意这个Done的位置，是另一个函数
            // 也可以将 defer wg.Done() 换成 defer wg.Add(-1)
            EchoNumber(n)
        }(i)
    }

    wg.Wait()
}

func EchoNumber(i int) {
    time.Sleep(3e9)
    fmt.Println(i)
}

```

golang中的同步是通过 `sync.WaitGroup` 来实现的。它实现了一个类似队列的结构，可以一直向队列中添加任务，当任务完成后便从队列中删除，如果队列中的任务没有完全完成，可以通过 `Wait()` 函数来出发阻塞，防止程序继续进行，直到所有的队列任务都完成为止．

WaitGroup的特点是Wait()可以用来阻塞直到队列中的所有任务都完成时才解除阻塞，而不需要sleep一个固定的时间来等待．但是其缺点是无法指定固定的goroutine数目．不过可以通过使用channel解决此问题。

**另一个例子**

```
package main

import (
    "fmt"
    "sync"
)

//声明一个全局变量
var waitgroup sync.WaitGroup

func Afunction(shownum int) {
    fmt.Println(shownum)
    defer waitgroup.Done() //任务完成，将任务队列中的任务数量-1，其实.Done就是.Add(-1)
}

func main() {
    for i := 0; i < 10; i++ {
        waitgroup.Add(1) //每创建一个goroutine，就把任务队列中任务的数量+1
        go Afunction(i)
    }
    waitgroup.Wait() //.Wait()这里会发生阻塞，直到队列中所有的任务结束就会解除阻塞
}
```