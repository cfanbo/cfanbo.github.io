---
title: Golang中的限速器 time/rate
author: admin
type: post
date: 2020-03-27T06:51:52+00:00
url: /archives/19884
categories:
 - 程序开发
tags:
 - golang

---
在高并发的系统中，限流已作为必不可少的功能，而常见的限流算法有：计数器、滑动窗口、令牌桶、漏斗(漏桶)。其中滑动窗口算法、令牌桶和漏斗算法应用最为广泛。

## 常见限流算法 

这里不再对 `计数器算法` 和 `滑动窗口` 算法一一介绍，有兴趣的同学可以参考其它相关文章。

### 漏斗算法 

漏斗算法很容易理解，它就像有一个漏斗容器一样，漏斗上面一直往容器里倒水(请求)，漏斗下方以**固定速率**一直流出(消费)。如果漏斗容器满的情况下，再倒入的水就会溢出，此时表示新的请求将被丢弃。可以看到这种算法在应对大的突发流量时，会造成部分请求弃用丢失。

可以看出漏斗算法能强行限制数据的传输速率。![](https://blog.haohtml.com/wp-content/uploads/2020/03/rate-limit.png)漏斗算法

### 令牌桶算法 

从某种意义上来说，令牌算法是对漏斗算法的一种改进。对于很多应用场景来说，除了要求能够限制数据的平均传输速率外，还要求允许某种程度的突发情况。这时候漏桶算法可能就不合适了，令牌桶算法更为适合。

令牌桶算法是指一个固定大小的桶，可以存放的令牌的最大个数也是固定的。此算法以一种**固定速率**不断的往桶中存放令牌，而每次请求调用前必须先从桶中获取令牌才可以。否则进行拒绝或等待，直到获取到有效令牌为止。如果桶内的令牌数量已达到桶的最大允许上限的话，则丢弃令牌。![](https://blog.haohtml.com/wp-content/uploads/2020/03/rate-limit-token-882x1024.png)

## Golang中的限制算法 

Golang标准库中的限制算法是基于令牌桶算法(Token Bucket) 实现的，库名为`golang.org/x/time/rate`

对于限流器的消费方式有三种，分别为 Allow()、 Wait()和 Reserve()。前两种内部调用的都是Reserve() ,每个都对应一个XXXN()的方法。如Allow()是AllowN(t, 1)的简写方式。

## 结构体 

```
type Limiter struct {
	limit Limit
	burst int
	mu     sync.Mutex
	tokens float64
	// last is the last time the limiter's tokens field was updated
	last time.Time
	// lastEvent is the latest time of a rate-limited event (past or future)
	lastEvent time.Time
}
```

主要用来限速控制并发事件，采用令牌池算法实现。

## 创建限速器 

使用 NewLimiter(r Limit, b int) 函数创建限速器，令牌桶容量为b。初始化状态下桶是满的，即桶里装有b 个令牌，以后再以每秒往里面填充 r 个令牌。

```
func NewLimiter(r Limit, b int) *Limiter {
	return &Limiter{
		limit: r,
		burst: b,
	}
}
```

允许声明容量为0的限速器，此时将会拒绝所有操作。

// As a special case, if r == Inf (the infinite rate), b is ignored.
有一种特殊情况，就是 r == Inf 时，此时b参数将被忽略。

```
// Inf is the infinite rate limit; it allows all events (even if burst is zero).
const Inf = Limit(math.MaxFloat64)
```

Limiter 提供了三个主要函数 Allow, Reserve, 和 Wait. 大部分时候使用Wait。其中 AllowN, ReserveN 和 WaitN 允许消费n个令牌。

每个方法都可以消费一个令牌，当没有可用令牌时，三个方法的处理方式不一样

 * 如果没有令牌时，Allow 返回 false。
 * 如果没有令牌时，Wait 会阻塞走到有令牌可用或者超时取消（context.Context)。
 * 如果没有令牌时，Reserve 返回一个 reservation，以便token的预订时，调用之前必须等待一段时间。

## 1. Allow/AllowN 

AllowN方法表示，截止在某一时刻，目前桶中数目是否至少为n个。如果条件满足，则从桶中消费n个token，同时返回true。反之不消费Token，返回false。

使用场景：一般用在如果请求速率过快，直接拒绝请求的情况

```
package main
import (
	"context"
	"fmt"
	"time"
	"golang.org/x/time/rate"
)
func main() {
	// 初始化一个限速器，每秒产生10个令牌，桶的大小为100个
	// 初始化状态桶是满的
	var limiter = rate.NewLimiter(10, 100)
	for i := 0; i < 20; i++ {
		if limiter.AllowN(time.Now(), 25) {
			fmt.Printf("%03d Ok  %sn", i, time.Now().Format("2006-01-02 15:04:05.000"))
		} else {
			fmt.Printf("%03d Err %sn", i, time.Now().Format("2006-01-02 15:04:05.000"))
		}
		time.Sleep(500 * time.Millisecond)
	}
}
```

输出

```
000 Ok  2020-03-27 16:17:18.604
001 Ok  2020-03-27 16:17:19.110
002 Ok  2020-03-27 16:17:19.612
003 Ok  2020-03-27 16:17:20.115
004 Err 2020-03-27 16:17:20.620
005 Ok  2020-03-27 16:17:21.121
006 Err 2020-03-27 16:17:21.626
007 Err 2020-03-27 16:17:22.127
008 Err 2020-03-27 16:17:22.632
009 Err 2020-03-27 16:17:23.133
010 Ok  2020-03-27 16:17:23.636
011 Err 2020-03-27 16:17:24.138
012 Err 2020-03-27 16:17:24.642
013 Err 2020-03-27 16:17:25.143
014 Err 2020-03-27 16:17:25.644
015 Ok  2020-03-27 16:17:26.147
016 Err 2020-03-27 16:17:26.649
017 Err 2020-03-27 16:17:27.152
018 Err 2020-03-27 16:17:27.653
019 Err 2020-03-27 16:17:28.156
```

## 2. Wait/WaitN 

当使用Wait方法消费Token时，如果此时桶内Token数量不足(小于N)，那么Wait方法将会阻塞一段时间，直至Token满足条件。否则直接返回。
// 可以看到Wait方法有一个context参数。我们可以设置context的Deadline或者Timeout，来决定此次Wait的最长时间。

```
func main() {
	// 指定令牌桶大小为5，每秒补充3个令牌
	limiter := rate.NewLimiter(3, 5)
	// 指定超时时间为5秒
	ctx, cancel := context.WithTimeout(context.Background(), time.Second*5)
	defer cancel()
	for i := 0; ; i++ {
		fmt.Printf("%03d %sn", i, time.Now().Format("2006-01-02 15:04:05.000"))
		// 每次消费2个令牌
		err := limiter.WaitN(ctx, 2)
		if err != nil {
			fmt.Printf("timeout: %sn", err.Error())
			return
		}
	}
	fmt.Println("main")
}
```

输出

```
000 2020-03-27 16:53:34.764
001 2020-03-27 16:53:34.764
002 2020-03-27 16:53:34.764
003 2020-03-27 16:53:35.100
004 2020-03-27 16:53:35.766
005 2020-03-27 16:53:36.434
006 2020-03-27 16:53:37.101
007 2020-03-27 16:53:37.770
008 2020-03-27 16:53:38.437
009 2020-03-27 16:53:39.101
timeout: rate: Wait(n=2) would exceed context deadline
```

## 3. Reserve/ReserveN 

// 此方法有一点复杂，它返回的是一个*Reservation类型，后续操作主要针对的全是这个类型
// 判断限制器是否能够在指定时间提供指定N个请求令牌。
// 如果Reservation.OK()为true，则表示需要等待一段时间才可以提供，其中Reservation.Delay()返回需要的延时时间。
// 如果Reservation.OK()为false,则Delay返回InfDuration, 此时不想等待的话，可以调用 Cancel()取消此次操作并归还使用的token

```
func main() {
	// 指定令牌桶大小为5，每秒补充3个令牌
	limiter := rate.NewLimiter(3, 5)
	// 指定超时时间为5秒
	ctx, cancel := context.WithTimeout(context.Background(), time.Second*5)
	defer cancel()
	for i := 0; ; i++ {
		fmt.Printf("%03d %sn", i, time.Now().Format("2006-01-02 15:04:05.000"))
		reserve := limiter.Reserve()
		if !reserve.OK() {
			//返回是异常的，不能正常使用
			fmt.Println("Not allowed to act! Did you remember to set lim.burst to be > 0 ?")
			return
		}
		delayD := reserve.Delay()
		fmt.Println("sleep delay ", delayD)
		time.Sleep(delayD)
		select {
		case <-ctx.Done():
			fmt.Println("timeout, quit")
			return
		default:
		}
		//TODO 业务逻辑
	}
	fmt.Println("main")
}
```

输出

```
000 2020-03-27 16:57:23.135
sleep delay  0s
001 2020-03-27 16:57:23.135
sleep delay  0s
002 2020-03-27 16:57:23.135
sleep delay  0s
003 2020-03-27 16:57:23.135
sleep delay  0s
004 2020-03-27 16:57:23.135
sleep delay  0s
005 2020-03-27 16:57:23.135
sleep delay  333.292866ms
006 2020-03-27 16:57:23.474
sleep delay  328.197741ms
007 2020-03-27 16:57:23.804
sleep delay  331.211817ms
008 2020-03-27 16:57:24.136
sleep delay  332.779335ms
009 2020-03-27 16:57:24.473
sleep delay  328.952586ms
010 2020-03-27 16:57:24.806
sleep delay  329.620588ms
011 2020-03-27 16:57:25.136
sleep delay  332.404798ms
012 2020-03-27 16:57:25.474
sleep delay  328.456103ms
013 2020-03-27 16:57:25.803
sleep delay  331.34754ms
014 2020-03-27 16:57:26.136
sleep delay  332.285545ms
015 2020-03-27 16:57:26.473
sleep delay  328.673618ms
016 2020-03-27 16:57:26.803
sleep delay  332.296438ms
017 2020-03-27 16:57:27.137
sleep delay  332.201646ms
018 2020-03-27 16:57:27.474
sleep delay  328.312813ms
019 2020-03-27 16:57:27.803
sleep delay  332.210098ms
020 2020-03-27 16:57:28.136
sleep delay  332.854719ms
timeout, quit
```

## 参考资料 
* https://www.cyhone.com/articles/analisys-of-golang-rate/
* https://zhuanlan.zhihu.com/p/100594314
* https://www.jianshu.com/p/1ecb513f7632
* https://studygolang.com/articles/10148