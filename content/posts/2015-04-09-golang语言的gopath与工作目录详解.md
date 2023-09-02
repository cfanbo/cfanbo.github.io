---
title: Golang语言的GOPATH与工作目录详解
author: admin
type: post
date: 2015-04-08T17:21:10+00:00
url: /archives/15567
categories:
 - 程序开发
tags:
 - golang
 - gopath

---
这篇文章主要介绍了Go语言的GOPATH与工作目录详解,本文详细讲解了GOPATH设置、应用目录结构、编译应用等内容,需要的朋友可以参考下

**GOPATH设置**

go 命令依赖一个重要的环境变量：$GOPATH

（注：这个不是Go安装目录( **GOROOT**)。下面以笔者的工作目录为说明，请替换自己机器上的工作目录。）


在类似 Unix 环境大概这样设置：

```
[shell]export GOPATH=/home/apple/mygo[/shell]
```

为了方便，应该把新建以上文件夹，并且把以上一行加入到 .bashrc 或者 .zshrc 或者自己的 sh 的配置文件中。


Windows 设置如下，新建一个环境变量名称叫做GOPATH：

```
[shell]GOPATH=c:mygo[/shell]
```

GOPATH允许多个目录，当有多个目录时，请注意分隔符，多个目录的时候Windows是分号，Linux系统是冒号，当有多个GOPATH时，默认会将go get的内容放在第一个目录下


**以上 $GOPATH 目录约定有三个子目录：**

1.src 存放源代码（比如：.go .c .h .s等）

2.pkg 编译后生成的文件（比如：.a）

3.bin 编译后生成的可执行文件（为了方便，可以把此目录加入到 $PATH 变量中，如果有多个gopath，那么使用${GOPATH//://bin:}/bin添加所有的bin目录）


以后我所有的例子都是以mygo作为我的gopath目录

**应用目录结构**

建立包和目录：$GOPATH/src/mymath/sqrt.go（包名：”mymath”）


以后自己新建应用或者一个代码包都是在src目录下新建一个文件夹，文件夹名称一般是代码包名称，当然也允许多级目录，例如在src下面新建了目录$GOPATH/src/github.com/astaxie/beedb 那么这个包路径就是“github.com/astaxie/beedb”，包名称是最后一个目录beedb


```
[shell]cd $GOPATH/src
mkdir mymath[/shell]
```

新建文件sqrt.go，内容如下：


```
[shell]
// $GOPATH/src/mymath/sqrt.go源码如下：
package mymath&amp;nbsp;&amp;nbsp;&amp;nbsp;

func Sqrt(x float64) float64 {
z := 0.0
for i := 0; i &amp;lt; 1000; i++ {
z -= (z*z - x) / (2 * x)
}
return z
}[/shell]
```

这样我的应用包目录和代码已经新建完毕，注意：一般建议package的名称和目录名保持一致


**编译应用**

上面我们已经建立了自己的应用包，如何进行编译安装呢？有两种方式可以进行安装


1、只要进入对应的应用包目录，然后执行go install，就可以安装了

2、在任意的目录执行如下代码go install mymath


安装完之后，我们可以进入如下目录：


```
[shell]cd $GOPATH/pkg/${GOOS}_${GOARCH}[/shell]
```

//可以看到如下文件

mymath.a


这个.a文件是应用包，那么我们如何进行调用呢？


接下来我们新建一个应用程序来调用


新建应用包mathapp：


```
[shell]cd $GOPATH/src
mkdir mathapp
cd mathapp
vim main.go[/shell]
```

// $GOPATH/src/mathapp/main.go源码：


```
[shell]package main&amp;nbsp;&amp;nbsp;&amp;nbsp;
import (
"mymath"
"fmt"
)

func main() {
fmt.Printf("Hello, world.&amp;nbsp; Sqrt(2) = %vn", mymath.Sqrt(2))
}[/shell]
```

如何编译程序呢？进入该应用目录，然后执行go build，那么在该目录下面会生成一个mathapp的可执行文件


```
[shell]./mathapp[/shell]
```

输出如下内容


```
[shell]Hello, world.&amp;nbsp; Sqrt(2) = 1.414213562373095[/shell]
```

如何安装该应用，进入该目录执行go install,那么在$GOPATH/bin/下增加了一个可执行文件mathapp,这样可以在命令行输入如下命令就可以执行


```
[shell]mathapp[/shell]
```

也是输出如下内容


```
[shell]Hello, world.&amp;nbsp; Sqrt(2) = 1.414213562373095[/shell]
```

**获取远程包**

go语言有一个获取远程包的工具就是go get，目前go get支持多数开源社区(例如：github、googlecode、bitbucket、Launchpad)


```
[shell]go get github.com/astaxie/beedb[/shell]
```

go get -u 参数可以自动更新包，而且当go get的时候会自动获取该包依赖的其他第三方包

通过这个命令可以获取相应的源码，对应的开源平台采用不同的源码控制工具，例如github采用git、googlecode采用hg，所以要想获取这些源码，必须先安装相应的源码控制工具


通过上面获取的代码在我们本地的源码相应的代码结构如下：


[![go_dir1](http://blog.haohtml.com/wp-content/uploads/2015/04/go_dir1.png)](http://blog.haohtml.com/wp-content/uploads/2015/04/go_dir1.png)

go get本质上可以理解为首先第一步是通过源码工具clone代码到src下面，然后执行go install


在代码中如何使用远程包，很简单的就是和使用本地包一样，只要在开头import相应的路径就可以


```
[shell]import "github.com/astaxie/beedb"[/shell]
```

**程序的整体结构**

通过上面建立的我本地的mygo的目录结构如下所示


![go_dir](http://blog.haohtml.com/wp-content/uploads/2015/04/go_dir.png)

从上面的结构我们可以很清晰的看到，bin目录下面存的是编译之后可执行的文件，pkg下面存放的是函数包，src下面保存的是应用源代码。[1] Windows系统中环境变量的形式为%GOPATH%，本书主要使用Unix形式，Windows用户请自行替换。