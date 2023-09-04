---
title: golang中flag包的用法
author: admin
type: post
date: 2015-06-09T16:57:44+00:00
url: /archives/15725
categories:
 - 程序开发
tags:
 - golang

---
golang中flag包主要用来CLI下，获取命令参数，示例如下mysql.go：

```
package main

import (
"flag"
"fmt"
)

func main() {
  host := flag.String("h", "localhost", "请指定一个主机")
  user := flag.String("u", "root", "请指定数据库用户")
  port := flag.Int("P", 3306, "Port number to use for commection or 0 for default to, in port 3306")

  //var name string
  //flag.StringVar(&name, "u", "root", "请指定用户名")

  flag.Parse() //参数解析

  fmt.Println("主机地址：", *host)
  fmt.Println("用户名：", *user)
  fmt.Println("端口:", *port)
}
```



像flag.Int、flag.Bool、flag.String这样的函数格式都是一样的，第一个参数表示参数名称，第二个参数表示默认值，第三个参数表示使用说明和描述。flag.StringVar这样的函数第一个参数换成了变量地址，后面的参数和flag.String是一样的。

使用flag来操作命令行参数，支持的格式如下：

```
go run mysql.go -h="127.0.0.1" -u="sxf" -P=3307
go run mysql.go --h="127.0.0.1" --u="sxf" --P=3307
go run mysql.go -h "127.0.0.1" -u "sxf"
go run mysql.go --h "127.0.0.1" --u "sxf" --P 3307
```

还是非常方便的。

这里不指定参数的情况，会使用默认值：

```
go run mysql.go
```

