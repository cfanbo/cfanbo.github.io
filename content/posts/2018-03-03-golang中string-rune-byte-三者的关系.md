---
title: golang中string rune byte 三者的关系
author: admin
type: post
date: 2018-03-03T04:54:59+00:00
url: /archives/17646
categories:
 - 程序开发
tags:
 - golang

---

`Go` 语言中 `byte` 和 `rune` 实质上就是 `uint8` 和 `int32` 类型。 `byte` 用来强调数据是 `raw data`，而不是数字；而 `rune` 用来表示 `Unicode` 的 `code point`。参考 [规范.](https://golang.org/ref/spec#Numeric_types)

在Golang中 string 底层是用byte字节数组存储的，并且是不可以修改的。

# Go语言中的byte和rune区别、对比 {.postTitle.wp-block-heading}

例如

```
s:="Go编程"
fmt.Println(len(s)) //输出结果应该是8因为中文字符是用3个字节存的(2+3*2=8)。
fmt.Printf("%d", len(string(rune('编')))) //经测试一个汉字确实占用3个字节，所以结果是3
```

如果想要获得字符个数的话，需要先转换为rune切片再使用内置的len函数

```
fmt.Println(len([]rune(s))) // 结果就是4了。
```

所以用string存储unicode的话，如果有中文，按下标是访问不到的，因为你只能得到一个byte。 要想访问中文的话，还是要用rune切片，这样就能按下表访问。

**总结：**
`rune` 能操作任何字符
`byte` 不支持中文的操作

示例： [https://blog.haohtml.com/archives/14903](https://blog.haohtml.com/archives/14903)

在极客时间的 **go语言核心36讲** 专栏里有一篇文章“ [unicode与字符编码](https://time.geekbang.org/column/article/64407)”对此介绍的比较详情，包含底层字符集编码。

> 在 Go 语言中，一个string类型的值既可以被拆分为一个包含多个字符的序列，也可以被拆分为一个包含多个字节的序列。前者可以由一个以 **rune** 为元素类型的切片来表示，而后者则可以由一个以 **byte** 为元素类型的切片代表。 rune是 Go 语言特有的一个基本数据类型，它的一个值就代表一个字符，即：一个 Unicode 字符。比如，’G’、’o’、’爱’、’好’、’者’代表的就都是一个 Unicode 字符。