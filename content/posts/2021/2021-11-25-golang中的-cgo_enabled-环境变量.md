---
title: Golang中的 CGO_ENABLED 环境变量
author: admin
type: post
date: 2021-11-25T04:40:58+00:00
url: /archives/31332
categories:
 - 程序开发
tags:
 - golang

---
Golang中的编译参数

开发中经常使用 `go build` 命令来编译我们的程序源码，然后将生成二进制文件直接部署，极其方便。

对于 `go build` 有一些参数，对于针对程序源码进行一些编译优化，下面我们对经常使用的一些参数来介绍一下。

# 环境变量

环境变量需要在go命令前面设置，如果多个变量的话，中间需要用“空格”分隔。下面我们介绍一个非常常见到的一些环境变量

```
$ CGO_ENABLED=1 GOARCH=amd64 GOOS=linux go build -o myserver main.go

```

除了这里给出的这几个变量外，还有一些其它变量，如 GODEBUG、GOFLAGS、GOPROXY 等，所有支持环境变量都可以在  里找到，有兴趣的话可以看看他们的作用。

这里重点介绍一下 `CGO_ENABLED` 环境变量对我们程序的影响。 `CGO_ENABLED`是用来控制golang 编译期间是否支持调用 cgo 命令的开关，其值为1或0，默认情况下值为1，可以用 `go env` 查看默认值。

如果你的程序里调用了cgo 命令，此参数必须设置为1,否则将编译时出错。这里直接用文档  中的一个例子验证。

```
package main

// #include <stdio.h>
// #include <stdlib.h>
//
// static void myprint(char* s) {
//   printf("%sn", s);
// }
import "C"
import "unsafe"

func main() {
	cs := C.CString("Hello from stdio")
	C.myprint(cs)
	C.free(unsafe.Pointer(cs))
}

```

然后我们执行一下 启用 CGO_ENABLED=1 的情况

```
root@ubuntu:/home/sxf/gotest# CGO_ENABLED=1 go run main.go
root@ubuntu:/home/sxf/gotest# ./main
Hello from stdio

```

可以看到输出正常，这里也可以省略不写变量，因为默认情况为启用CGO状态

启用 CGO_ENABLED=0 的情况

```
root@ubuntu:/home/sxf/gotest# CGO_ENABLED=0 go build main.go
go: no Go source files

```

可以看到编译失败，验证了我们上面说的情况。

这里提示找不到go源文件，不清楚底层是如何判断这一情况的。

那么，如果我们一个程序里未调用cgo，在编译时指定 `CGO_ENABLED` 不同值话，又会发生什么呢？编译的二进制有何区别呢？

```
root@ubuntu:/home/sxf/gotest# CGO_ENABLED=1 go build -o cgo_main main.go
root@ubuntu:/home/sxf/gotest# CGO_ENABLED=0 go build -o no_cgo_main main.go
root@ubuntu:/home/sxf/gotest# ls -al
total 3804
drwxrwxr-x  2 sxf  sxf     4096 Nov 25 10:22 .
drwxr-x--- 47 sxf  sxf     4096 Nov 19 12:48 ..
-rwxr-xr-x  1 root root 1937311 Nov 25 10:22 cgo_main
-rw-rw-r--  1 sxf  sxf      119 Oct 29 09:07 go.mod
-rw-rw-r--  1 sxf  sxf      241 Oct 29 09:07 go.sum
-rw-rw-r--  1 sxf  sxf       72 Nov 25 10:21 main.go
-rwxr-xr-x  1 root root 1937311 Nov 25 10:22 no_cgo_main

```

可以看到两种情况都可能编译成功，且两者生成的二进制文件是完全一样的。由此可以看出如果程序里未调用 cgo 的话，此变量值并不会产生任何影响。

那么问题来了，为什么这么多项目里编译的时候都明确指定了此环境变量的值呢，主要是编译器在编译时会根据不同的情况使用不同的编译方法。 当指定 `CGO_ENABLED=1` 编译时, 会将文件中引用 `libc` 的库（比如常用的 `net` 包），以动态链接的方式生成目标文件。 当指定`CGO_ENABLED=0` 编译时，则会把在目标文件中未定义的符号（外部函数）一起链接到可执行文件中。

不论哪种方式，都可以使用静态连接编译

# 参考 
* https://go.dev/blog/cgo
* https://pkg.go.dev/cmd/cgo
* https://johng.cn/cgo-enabled-affect-go-static-compile/
* https://github.com/golang/go/blob/a040ebeb98/src/cmd/cgo/doc.go
* https://www.jianshu.com/p/bc78c32db030