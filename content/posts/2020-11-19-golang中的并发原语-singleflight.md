---
title: Golang中的并发原语 Singleflight
author: admin
type: post
date: 2020-11-19T11:37:27+00:00
url: /archives/20279
categories:
 - 程序开发
tags:
 - golang
 - Singleflight

---
在Golang中有一个并发原语是 [Singleflight](https://pkg.go.dev/golang.org/x/sync/singleflight)，好像知道的开发者并不多。其中著名的 [https://github.com/golang/groupcache](https://github.com/golang/groupcache) 就用到了这个并发原语。

## Golang版本 

go1.15.5

## 相关知识点 

map、Mutex、channel、

## 使用场景 

一般用在对指定资源频繁操作的情况下，如高并发下的“缓存击穿”问题。

> 缓存击穿：一个存在的key，在缓存过期的瞬间，同时有大量的请求过来，造成所有请求都去DB读取数据，这些请求都会击穿缓存到DB，造成瞬时DB请求量大、压力瞬间骤增，导致数据库负载过高，影响整个系统正常运行。（缓存击穿不同于 缓存雪崩 和 缓存穿透）

怎么理解这个原语呢，简单的讲就是将对同一个资源的多个请求合并为一个请求。

举例说明，假如当有10万个请求来获取同一个key的值的时候，正常情况下会执行10万次get操作。而使用singleflight并发语后，只需要首次的地个请求执行一次get操作就可以了，其它请求再过来时，只需要只需要等待即可。待执行结果返回后，再把结果分别返回给等待中的请求，每个请求再返回给客户端，由此看看，在一定的高并发场景下可以大大减少系统的负载，节省大量的资源。

注意这个与 sync.Once是不一样的，sync.Once 是全局只能有一个，但本并发原语则是根据key来划分的，并且可以根据需求来决定什么情况下共用一个。

## 实现原理 

主要使用 Mutext 和 Map 来实现，以 key 为键，值为 \*call。每个\*call中存储有一个请求 chans 字段，用来存储所有请求此key的客户端，等有返回结果的时候，再从chans字段中读取出来，分别写入即可。

源文件为 /src/internal/singleflight/singleflight.go

Singleflight 数据结构如下![a9d00f0444df6bf47e2aa089eca61e44](https://blogstatic.haohtml.com/uploads/2020/11/37a445d94ec4f80d21a7f3f5022a9569-1.jpg)

 1. **Do()**
 这个方法是一个执行函数并返回执行结果
 参数
 `key` 要请求的key，多个请求可能请求的是同一个key，同时也只有一个函数在执行
 `fn` 对应执行函数，此函数有三个返回值`v`, `err`, `shared`。其中`shared`表示当前返回结果是否为多个请求结果
 2. **DoChan()**
 类型与`Do()`方法，但返回的是个 ch 类型，等函数 fn 执行后，可以通过读取返回的ch来获取函数结果
 3. **Forget()**
 在官方库`internal/singleflight/singleflight.go`中这个名字是**ForgetUnshared**， 告诉 Group 忘记请求的这个key，下次再请求时，直接当作新的key来处理就可以了。其实就是将这个key从map中删除，后续再有这个key的操作的话，即视为新一轮的处理逻辑。

其中`Do()` 和 `DoChan()` 的功能一样，只是获取数据的方式有所区别，开发者可以根据自己的情况来选择使用哪一个。另外还包含一个由Do()方法调用的私有方法 doCall()，直接执行key处理方法的函数

```
func (g *Group) doCall(c *call, key string, fn func() (interface{}, error)) {...}

```

## 实现数据结构 

实现内部主要的数据结构一共三个，分别是 call、Group 和 Result。

```
// 定义 map[key] 的值 call 类型
// 在call 内部存储对应key相关的所有请求客户端信息
type call struct {
	wg sync.WaitGroup
	// These fields are written once before the WaitGroup is done
	// and are only read after the WaitGroup is done.
        // 请求返回结果，会在sync.WaitGroup为Done的时候执行
	val interface{}
	err error
	// These fields are read and written with the singleflight
	// mutex held before the WaitGroup is done, and are read but
	// not written after the WaitGroup is done.
        // 可以理解为请求key的个数，每增加一个请求，则值加1
	dups  int
        // 存储所有key对应的 Result{}, 一个请求对应一个 Result
	chans []chan<- Result
}
// Group represents a class of work and forms a namespace in
// which units of work can be executed with duplicate suppression.
// SingleFlight 的主结构体
type Group struct {
	mu sync.Mutex       // protects m
	m  map[string]*call // lazily initialized
}
// Result holds the results of Do, so they can be passed
// on a channel.
// 定义请求结果数据结构
type Result struct {
	Val    interface{}
	Err    error
	Shared bool
}
```

**call** 数据结构是用来记录key相关数据，即请求当前key的个数和请求结果
**Group** 是并发原语 SingleFlight 的主数据结构, m 字段用来存储key与请求的关系，而mu则是Mutex锁
**Result** 请求结果，其中 `Val` 是实现请求后返回的结果，`Err` 表示是否出错，而 `Shared` 字段则表示是否为多个请求结果。如果当前key只有一个请求的话，则返回false, 否则返回 true

## 用法 

```
package main
import (
	"fmt"
	"golang.org/x/sync/singleflight"
	"log"
	"sync"
	"time"
)
func getDataFromDB(key string) (string, error) {
	defer func() {
		log.Println("db query end")
	}()
	log.Println("db query begin")
	time.Sleep(2 * time.Second)
	result := key + "abcxyz"
	return result, nil
}
func main() {
	var singleRequest singleflight.Group
	getData := func(requestID int, key string) (string, error) {
		log.Printf("request %v start request ...", requestID)
		// 合并请求
		value, _, _ := singleRequest.Do(key, func() (ret interface{}, err error) {
			log.Printf("request %v is begin...", requestID)
			ret, err = getDataFromDB(key)
			log.Printf("request %v end!", requestID)
			return
		})
		return value.(string), nil
	}
	var wg sync.WaitGroup
	key := "orderID"
	for i := 1; i < 10; i++ {
		wg.Add(1)
		go func(wg *sync.WaitGroup, requestID int) {
			defer wg.Done()
			value, _ := getData(requestID, key)
			log.Printf("request %v get value: %v", requestID, value)
		}(&wg, i)
	}
	wg.Wait()
	fmt.Println("main end")
}

```

输出结果

```
2020/11/20 12:11:44 request 6 start request …
2020/11/20 12:11:44 request 6 is begin…
2020/11/20 12:11:44 request 2 start request …
2020/11/20 12:11:44 request 9 start request …
2020/11/20 12:11:44 request 7 start request …
2020/11/20 12:11:44 request 3 start request …
2020/11/20 12:11:44 request 4 start request …
2020/11/20 12:11:44 request 1 start request …
2020/11/20 12:11:44 request 5 start request …
2020/11/20 12:11:44 request 8 start request …
2020/11/20 12:11:44 db query begin
2020/11/20 12:11:46 db query end
2020/11/20 12:11:46 request 6 end!
2020/11/20 12:11:46 request 6 get value: orderIDabcxyz
2020/11/20 12:11:46 request 8 get value: orderIDabcxyz
2020/11/20 12:11:46 request 2 get value: orderIDabcxyz
2020/11/20 12:11:46 request 9 get value: orderIDabcxyz
2020/11/20 12:11:46 request 7 get value: orderIDabcxyz
2020/11/20 12:11:46 request 3 get value: orderIDabcxyz
2020/11/20 12:11:46 request 4 get value: orderIDabcxyz
2020/11/20 12:11:46 request 1 get value: orderIDabcxyz
2020/11/20 12:11:46 request 5 get value: orderIDabcxyz
main end
```

从上面的输出内容可以看到，我们同时发起了10个gorourtine来查询同一个订单信息，真正在DB 层查询操作只有一次，大大减少了DB的压力。

## 总结 

使用SingleFlight时，在高并发下且对少量key频繁读取的话，可以大大减少服务器负载。在一定场景下特别的有用，如 CDN。

**推荐** [Golang并发原语之-信号量Semaphore](https://blog.haohtml.com/archives/25563)