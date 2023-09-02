---
title: Sublime Text 2搭建Go开发环境（Windows）
author: admin
type: post
date: 2013-05-25T04:15:09+00:00
url: /archives/13836
categories:
 - 程序开发
tags:
 - golang
 - sublime

---
Go是Google开发的一种编译型，并发型，并具有垃圾回收功能的编程语言。

罗伯特·格瑞史莫（Robert Griesemer），罗勃·派克（Rob Pike）及肯·汤普逊于2007年9月开始设计Go语言，Go语言是基于Inferno操作系统所开发的。Go语言于2009年11月正式宣布推出，并在Linux及Mac OS X平台上进行了实现.

GO语言吉祥物,很可爱吧。

![](http://my.csdn.net/uploads/201207/17/1342492742_5865.jpg)

Go语言的hello world！代码:

[java]package main


import "fmt"


func main() {

fmt.Println("Hello, 世界")

}[/java]


接下来为大家带来，Go开发环境的安装。

首先是_**安装Go**_，这里有很详细的安装说明， [http://code.google.com/p/golang-china/wiki/Install](http://code.google.com/p/golang-china/wiki/Install) 或者 [http://golang.org/doc/install](http://golang.org/doc/install), windows环境参考： [http://blog.haohtml.com/archives/13527](http://blog.haohtml.com/archives/13527)

> **配置windows环境变量：**
>
> (1). 新建系统变量 变量名： GOROOT 变量值：d:\go
>
> (2). 新建系统变量 变量名：GOBIN 变量值 ：%GOROOT%\bin
>
> (3). 编辑系统变量 Path 在Path的变量值的最后加上 ;%GOBIN%

下面我们在window下面安装，google有提供win安装包，对于新手还是非常简单的！

[https://code.google.com/p/go/downloads/list](https://code.google.com/p/go/downloads/list)

直接下一步…….安装非常简单！

安装好Go以后，我们就可以_**搭建开发环境**_了，这里我用的是 Sublime Text 2 + GoSublime + gocode。对于不了解Sublime Text 2的朋友，可以看看官网: [http://www.sublimetext.com/](http://www.sublimetext.com/)(总的来说是一个轻量级，用起来很方便的工具)

1. 下载 Sublime Text 2(下载32位的，下面有破解方法，64位的无法使用下面的破解)，地址如下：

2. 解压以后，双击 sublime_text，就可以使用 Sublime Text 2 了。

**破解方法：用 WinHex 编辑 sublime\_text\_backup.exe 文件,** 跳到 **000CBB70** 那一行，将该行的 **8A** C3 修改为 B0 01 然后保存重新打开软件即可.**十六进制编辑器下载:** [WinHexGreen.zip](http://www.xiazaiba.com/html/427.html)

3. 安装 [Package Control][1]，在打开 Sublime Text 2以后，按下快捷键 **Ctrl + \`**，打开命令窗行，｀这个按键在Tab键的上面，我刚开始还没找到，呵呵。输入以下内容，并回车：

> import urllib2,os; pf=’Package Control.sublime-package’; ipp=sublime.installed\_packages\_path(); os.makedirs(ipp) if not os.path.exists(ipp) else None; urllib2.install\_opener(urllib2.build\_opener(urllib2.ProxyHandler())); open(os.path.join(ipp,pf),’wb’).write(urllib2.urlopen(‘http://sublime.wbond.net/’+pf.replace(‘ ‘,’%20’)).read()); print ‘Please restart Sublime Text to finish installation’

4. 重启Sublime Text 2后，就可以发现在 **Preferences** 菜单下，多出一个菜单项 **Package Control**。

5.现在安装GoSublime插件了，按住**Ctrl+Shilft+p**会弹出一个对话框

![](http://my.csdn.net/uploads/201207/17/1342495258_3820.png)

输入install回车弹出一个安装包的对话框

![](http://my.csdn.net/uploads/201207/17/1342495553_6970.png)

如入GoSublime选择 **GoSublime **回车

输入**Go build** 选中回车（这个属于可选）

搞定，GoSublime安装成功。

6.下面安装[gocode][2]

打开控制台(ctrl+b)，输入以下内容：

[java]go get github.com/nsf/gocode
go install github.com/nsf/gocode[/java]

也可以去github下载 [https://github.com/nsf/gocode.git](https://github.com/nsf/gocode.git)（要安装google的git版本管理工具）

安装完成后，我们可以在 go/bin 目录下，发现多出了个 gocode 文件。（一定要放在bin目录下）

7. 修改GoSublime配置：在 Preferences菜单下，找到Package Settings，然后找到 GoSublime，再往下找到 Settings – Default。再打开的文件中，添加如下配置，并保存：

![](http://my.csdn.net/uploads/201207/17/1342494236_3173.png)

好了，到目前为止，开发环境搭建完成。

打开 Sublime Text 2，新建 helloworld.go，编写代码如下：

见证Go代码自动提示的时刻了

输入一个p

![](http://my.csdn.net/uploads/201207/17/1342495981_5526.png)

回车（enter键）

![](http://my.csdn.net/uploads/201207/17/1342496011_9195.png)

main方法，包自动给你生成了。

下面是一个打印的例子：

![](http://my.csdn.net/uploads/201207/17/1342495860_4302.png)

按下快捷键 **Ctrl + b** 界面下方会出现如下界面：

![](http://my.csdn.net/uploads/201207/17/1342496239_5991.png)

输入 go build hello.go

![](http://my.csdn.net/uploads/201207/17/1342496289_4284.png)

运行，同样 按下快捷键 **Ctrl + b** 界面下方会出现如下界面，输入 hello回车 。如图：

![](http://my.csdn.net/uploads/201207/17/1342496342_6597.png)

好了，到现在，开发环境就搭建完毕了，希望大家也来学习Go这门语言。

新手入门参考:

[新手入门api](http://mikespook.com/learning-go) 是我见过的最好的新手入门文档,希望go能发扬光大。

 [1]: http://wbond.net/sublime_packages/package_control
 [2]: https://github.com/nsf/gocode