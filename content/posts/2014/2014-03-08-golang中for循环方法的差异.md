---
title: golang中for循环方法的差异
author: admin
type: post
date: 2014-03-08T08:25:11+00:00
url: /archives/14903
categories:
 - 程序开发
tags:
 - golang

---
用for循环遍历字符串时,也有 byte 和 rune 两种方式.第一种试为byte，第二种rune。

golang中string rune byte 三者的关系 [https://blog.haohtml.com/archives/17646](https://blog.haohtml.com/archives/17646)

```
package main

import (
    "fmt"
)

func main() {
    s := "abc汉字"

    for i := 0; i < len(s); i++ {
    fmt.Printf("%c,", s[i])
    }

    fmt.Println()

    for _, r := range s {
        fmt.Printf("%cn", r)
    }
}
```

输出结果:

```
a,b,c,æ,±,,å,­,,
a
b
c
汉
字

```