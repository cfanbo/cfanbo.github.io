---
title: golang中chan实例
author: admin
type: post
date: 2015-06-15T20:45:59+00:00
url: /archives/15762
categories:
 - 程序开发
tags:
 - golang

---

```
package main

import "fmt"

func main() {
 data := make(chan int) // 数据交换队列
 exit := make(chan bool) // 退出通知

go func() {
 for d := range data { // 从队列迭代接收数据,直到 close 。
   fmt.Println(d)
 }

 fmt.Println("recv over.")
 exit <- true // 发出退出通知。
}()

data <- 1 // 发送数据。
data <- 2
data <- 3

close(data) // 关闭队列。

fmt.Println("send over.")

<-exit // 等待退出通知。
}
```

输出结果：

```
1
2
3
send over.
recv over.

```

而如果将上面与 exit chan有关的三行删除掉，则结果为:

```
1
2
3
send over.
```

缺少了“recv over."一行，为什么？

大家可以  time.Sleep(time.Second * 2) 来自己分析一下