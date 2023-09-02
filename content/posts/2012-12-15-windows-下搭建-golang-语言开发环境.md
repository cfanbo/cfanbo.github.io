---
title: windows 下搭建 GoLang 语言开发环境
author: admin
type: post
date: 2012-12-15T05:18:47+00:00
url: /archives/13527
categories:
 - 程序开发
tags:
 - golang

---
golang官方二进制分发包包括FreeBSD, Linux, Mac OS X (Snow Leopard/Lion), and Windows等平台，包括32位、64位等版本。

我自己使用的是windows 32位分发包，MSI格式的，下载地址为： [http://code.google.com/p/go/downloads/list](http://code.google.com/p/go/downloads/list)

golang支持交叉编译，也就是说你在32位平台的机器上开发，可以编译生成64位平台上的可执行程序。

**环境变量说明：**
$GOROOT  指向golang安装之后的根目录，windows平台下默认为c:\go，会在安装过程中由安装程序自动写入系统环境变量。
$GOARCH  目标平台（编译后的目标平台）的处理器架构（386、amd64、arm）
$GOOS     目标平台（编译后的目标平台）的操作系统（darwin、freebsd、linux、windows）
$GOBIN     指向安装之后根目录下的bin目录，即$GOROOT/bin，windows平台下默认为c:\go\bin，会在安装过程中由安装程序自动添加到PATH变量中

**配置windows环境变量：**

(1). 新建系统变量 变量名： GOROOT 变量值：d:\go

(2). 新建系统变量 变量名：GOBIN 变量值 ：%GOROOT%\bin

(3). 编辑系统变量 Path 在Path的变量值的最后加上 ;%GOBIN%

(4). 确定保存变量配置

测试安装结果：
创建hello.go文件，内容如下：

```
package main
import "fmt"
func main() {
fmt.Printf(“hello, world\n”)
}

```

执行命令并显示结果
D:\godev>go run hello.go
hello, world
如果正确输出结果，则表明安装成功。接下来就可以使用了。

=================================================

首先从网上下载 windows golang 环境

64 和 32 分别下载 amd64 和 386的 压缩包。

我的电脑是 64 bit windows 7 所以下载

[gowinamd64_weekly.2012-01-15.zip](http://code.google.com/p/gomingw/downloads/detail?name=gowinamd64_weekly.2012-01-15.zip&can=2&q=)

这个事每周 打一个版本的。。更新速度还是挺快的。

然后解压缩到 d:/soft/go/目录下

然后安装 eclipse go 插件：

更新重启 eclipse 然后配置 golang 目录：

[![goclipse](http://blog.haohtml.com/wp-content/uploads/2012/12/goclipse.jpg)][1]

创建一个工程。写一个helloworld 如下：

[![eclipse_golang](http://blog.haohtml.com/wp-content/uploads/2012/12/eclipse_golang.jpg)][2]

都可以了。然后就可以开发 自己的应用了。

继续研究中。

 [1]: http://blog.haohtml.com/wp-content/uploads/2012/12/goclipse.jpg
 [2]: http://blog.haohtml.com/wp-content/uploads/2012/12/eclipse_golang.jpg