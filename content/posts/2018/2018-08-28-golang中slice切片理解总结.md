---
title: golang中slice切片理解总结
author: admin
type: post
date: 2018-08-28T02:34:32+00:00
url: /archives/18094
categories:
 - 程序开发
tags:
 - golang

---
首先我们对切片有一个大概的理解，先看一下slice的内部结构，共分三部分，一个是指向底层数组的时候，一个是长度len，另一个就是slice的容量cap了。如cap不足以放在新值的时候，会产生新的内存地址申请。

![image-20230904182516517](https://blog--static.oss-cn-shanghai.aliyuncs.com//uploads/2023/09/image-20230904182516517.png)



![image-20230904182527333](https://blog--static.oss-cn-shanghai.aliyuncs.com//uploads/2023/09/image-20230904182527333.png)

先看代码

```
package main

import "fmt"

func main() {

    // 创建一个切片，长度为9，容量为10
    fmt.Println("----- 1.测试切片变量append的影响（未申请新的内存空间）-----")
    a := make([]int, 9,10)
    fmt.Printf( "%p len=%d cap=%d %vn" , a, len(a), cap(a), a)

    // 切片进行append操作，由于原来len(a)长度为9,而cap(a)容量为10，未达到扩展内存的要求，此时新创建的切片变量还指向原来的底层数组，只是数组的后面添加一个新值
    // 此时一共两个切片变量，一个是a，另一个是s4。但共指向的一个内存地址
    s4 := append(a,4)
    fmt.Printf("%p len=%d cap=%d %vnn" , s4, len(s4), cap(s4), s4)

    // 测试上面提到的切片变量a和s4共指向同一个内存地址, 发现切片数组的第一个值都为7，而唯一不同的是len的长度，而cap仍为10
    fmt.Println("----- 2.测试切片变量共用一个底层数组（内存地址一样）-----")

    a[0] =  7
    fmt.Printf("%p len=%d cap=%d %vn" , a, len(a), cap(a), a)
    fmt.Printf("%p len=%d cap=%d %vnn" , s4, len(s4), cap(s4), s4)

    // 切片进行append操作后，发现原来的cap(a)的长度已用完了(因为a和s4共用一个底层数组，你也可以理解为cap(s4))，此时系统需要重新申请原cap*2大小的内存空间,所以cap值为10*2=20，将把原来底层数组的值复制到新的内存地址

    // 此时有两个底层数组，一个是切片变量a和s4指向的数组，另一个就是新的切片变量s4
    fmt.Println("----- 3.测试切片变量append的影响（申请了新的内存空间，内存地址不一样了）-----" )
    s4 = append(s4,  5 )
    fmt.Printf("%p len=%d cap=%d %vnn" , s4, len(s4), cap(s4), s4)

    // 注意：原切片未发生任何变化，(打印a[0]=7是因为上面第3段落代码已把默认的0值改为了7）
    fmt.Println("----- 4.测试原切片变量a未发生变化-----" )
    fmt.Printf("%p len=%d cap=%d %vnn" , a, len(a), cap(a), a)

}
```
运行结果：

```
----- 1.测试切片变量append的影响（未申请新的内存空间）-----
0x10450030 len=9 cap=10 [0 0 0 0 0 0 0 0 0]
0x10450030 len=10 cap=10 [0 0 0 0 0 0 0 0 0 4]

----- 2.测试切片变量共用一个底层数组-----
0x10450030 len=9 cap=10 [7 0 0 0 0 0 0 0 0]
0x10450030 len=10 cap=10 [7 0 0 0 0 0 0 0 0 4]

----- 3.测试切片变量append的影响（申请了新的内存空间）-----
0x10458050 len=11 cap=20 [7 0 0 0 0 0 0 0 0 4 5]

----- 4.测试原切片变量a未发生变化-----
0x10450030 len=9 cap=10 [7 0 0 0 0 0 0 0 0]
```


在线运行代码： [https://play.golang.org/p/pfxZa8T0H1_g](https://play.golang.org/p/pfxZa8T0H1_g)

当对slice进行append的时候，会出现以下情况：
**1.如果cap超出了原slice的cap长度**，则申请原来cap*2的内存空间，再把原来切片所指的底层数组的值复制一份存储到新申请的内存空间里，这样是两块内存地址了，所有原来slice指的底层数组内容没有变化的。此时两个切片，两个底层数组了，每个切片有自己对应的底层数组。
**2.如果cap没超出原slice的cap的话**，底层对应的数组是没有变化的，但会产生一个新的slice变量，仍然会指向到原来的底层数组（这个新slice变量有自己的内存地址，我们不用关心它，只需要关心他对应的底层数组就可以了）。这时一共两个切片，但共用一个底层数组。

注意本示例中每次 append 都是一个元素，如果是多个元素是否与上面的结论相符呢？ [https://play.studygolang.com/p/87HEvjPu7Lq](https://play.studygolang.com/p/87HEvjPu7Lq)

```
package main

import (
    "fmt"
)

func main() {
    a := []int{1, 2}
    a = append(a, 3, 4, 5)

    fmt.Printf("%p len=%d cap=%d %vn", a, len(a), cap(a), a)
}
```

执行结果里的cap会是多少呢？5 还是8？其实结果为 **6** 。主要是golang 为了节省内容，在上面扩容大小后，还需要进行二次检查，以达到减少内存浪费的效果。

主要相关函数是 [roundupsize()][3] 和 [divRoundUp()][4]，还有一些数组字典，根据计算结果选择不同大小的内存使用，可参考 [src/runtime/sizeclasses.go](https://github.com/golang/go/blob/go1.15.6/src/runtime/sizeclasses.go)

**总结：**
1.当对slice进行append的时候，如果原slice可以存放下新增加的值，则不会申请新的内存空间，否则会申请**cap*2**大小基准申请内存空间。而当cap>=1024的时候，会以**cap*1.25**倍的大小空间基准申请内存空间。
2.每次切片进行append操作，都会返回一个新的切片变量，这个变量有自己的内存地址。 这点开发者知道就可以了，一般不用关心。
3.当切片进行append后，原切片对应的底层数组是不会变化的，这点可从第4行的打印结果看到。

请参考golang开发源码包 src/runtime/slice.go 中的**growslice**函数，可以看到对于cap的增长处理逻辑。

[3]: https://github.com/golang/go/blob/go1.15.6/src/runtime/msize.go#L12-L25
[4]: https://github.com/golang/go/blob/go1.15.6/src/runtime/stubs.go#L313-L318