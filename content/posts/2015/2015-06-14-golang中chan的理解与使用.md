---
title: golang中chan的理解与使用教程
author: admin
type: post
date: 2015-06-13T22:34:52+00:00
url: /archives/15742
categories:
 - 程序开发
tags:
 - golang

---
对于 chan 介绍见： [https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.7.md](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.7.md)

这里我们主要通过实例来介绍对chan的理解及用法.

# 无Buffer的Channels

实例1：

```
func main() {
ci := make(chan int)

ci <- 4

value := <-ci
fmt.Println(value)
}

```

执行结果错误为：

> fatal error: all goroutines are asleep - deadlock!
> goroutine 1 [chan send]:

从上面“fatal error: all goroutines are asleep - deadlock!” 这句我们可以看出是groutings 阻塞了，这里为写阻塞，从“goroutine 1 [chan send]”可以看出来。

这一点文档里已经说明阻塞的原因了：

> 默认情况下，channel接收和发送数据都是阻塞的，除非另一端已经准备好，这样就使得Goroutines同步变的更加的简单，而不需要显式的lock。所谓阻塞，也就是如果读取（value := <-ch）它将会被阻塞，直到有数据接收。其次，任何发送（ch<-5）将会被阻塞，直到数据被读出。无缓冲channel是在多个goroutine之间同步很棒的工具。

**解决办法：**

我们只要将其中发送方(写)或者读取方(读)放到一个goroutine里就可以了，这样主程序main()里与goroutine通过channel来通讯即可。

```
func main() {
ci := make(chan int)

go write(ci)

value := <-ci
fmt.Println(value)
}

func write(c chan int) {
c <- 4
}

```

\---\---\---\---

# Buffered Channels

我们上面用到的是无buffer的channel, 这里我们使用带buffer的Channel来解决上面的问题。

# **目录：**

**  一. 不使用goroutine**

 1. **buffer size > 写的次数 (正常)**
 2. **buffer size = 写的次数 (正常)**
 3. **buffer size < 写的次数 (出错)
**

**  二. 使用goroutine**

 1. **buffer size > 写的次数 (正常)**
 2. **buffer size = 写的次数 (正常)**
 3. **buffer size < 写的次数 (正常)**

\---\---\---\---\---\---\---\---\---\---\---\---\---\---\----

**不使用goroutine的情况**

我们只要将上面的代码里“ci := make(chan int)” 指定一下buffer的大小为1即可， ci := make(chan int，1), 完整代码如下：

```
func main() {
ci := make(chan int, 1)

ci <- 4

value := <-ci
fmt.Println(value)
}

```

那么这里的buffer大小与实际对buffer操作(写)有什么关系呢，下面我们来测试一下。

**1.buffer size > 写的次数 (正常)**

先将buffer大小改为2,再次运行程序

```
func main() {
ci := make(chan int, 2)

ci <- 4

value := <-ci
fmt.Println(value)
}

```

程序输出与上方的完全一样，没有一点问题，原因见最上方链接对buffer channels的介绍：

> Go允许指定channel的缓冲大小，很简单，就是channel可以存储多少元素。ch:= make(chan bool, 4)，创建了可以存储4个元素的bool 型channel。在这个channel 中，前4个元素可以无阻塞的写入。当写入第5个元素时，代码将会阻塞，直到其他goroutine从channel 中读取一些元素，腾出空间。

**2. buffer size = 写的次数 (正常)**

那么我们这里buffer的大小比写的次数要大，如果次数完全一要，又会是什么结果呢？我们现在修改一下。

```
func main() {
ci := make(chan int, 2)

ci <- 4
ci <- 5

value := <-ci
value1 := <-ci

fmt.Println(value, value1)
}

```

现在执行一下，结果如下：

> 4 5

由此可以看出，当buffer的size  >= 写的次数 时，程序完全正常，而当 size < 写的次数时，又会怎样呢？

**3. buffer size < 写的次数 (出错)**

下面我们再测试一下buffer size < 写的次数的情况下，这里写入三个元素，buffer大小为2。

```
func main() {
ci := make(chan int, 2)

ci <- 4
ci <- 5
ci <- 6

value := <-ci
value1 := <-ci
value2 := <-ci

fmt.Println(value, value1, value2)
}

```

执行如何以下：

> fatal error: all goroutines are asleep - deadlock!
> goroutine 1 [chan send]:

可以看出来和我们第一次的出错信息完全一样，也是写阻塞了。国为这里buffer的大小为2,小于写的次数，buffer已满又没有对chan进行消费(< chan)，所以在ci <- 6 这句阻塞了。

由此我们可以确定对chan的使用中，一定要注意buffer的大小问题，但细心的你可能为发现这里并没有用到goroutine的，如果使用goroutine的话，这样写法是否还正确呢？**使用goroutine的情况**

下面我们和上面一样，分三种情况，分别是 buffer size = write次数、buffer size > write次数 和 buffer size < write次数。这里我们为了测试方便，修改了下write()函数.

```
func write(c chan int, i int) {
c <- i
}

```

**1.buffer size = write次数 (正常)**

```
func main() {
ci := make(chan int, 2)
go write(ci, 4)
go write(ci, 5)

value := <-ci
value1 := <-ci

fmt.Println(value, value1)
}

```

程序输出正常

> 4 5

**2.buffer size > write次数 (正常)**

```
func main() {
ci := make(chan int, 3)
go write(ci, 4)
go write(ci, 5)

value := <-ci
value1 := <-ci

fmt.Println(value, value1)
}
```

输出结果同上，正常。

**3.buffer size < write次数 (正常)**

```
func main() {
ci := make(chan int, 1)
go write(ci, 4)
go write(ci, 5)

value := <-ci
value1 := <-ci

fmt.Println(value, value1)
}

```

输出结果同上，仍然正常。

由此可以看出当buffer channels使用goroutine的时候，不存在buffer size 与次数关系的问题，因为仍是主函数main通过channel来和goroutine进行了通讯！

**总结：**

**在不使用grouting的情况下，如果buffer size < 写的次数的话，则程序并出错！**