---
title: Golang import使用说明
author: admin
type: post
date: 2013-10-31T02:49:50+00:00
url: /archives/14628
categories:
 - 程序开发
tags:
 - golang

---
我们在写Go代码的时候经常用到import这个命令用来导入包文件，而我们经常看到的方式参考如下：

```
[java]
import(
"fmt"
)[/java]
```

然后我们代码里面可以通过如下的方式调用

```
[java]fmt.Println("hello world")[/java]
```

上面这个fmt是Go语言的标准库，他其实是去goroot下去加载该模块，当然Go的import还支持如下两种方式来加载自己写的模块：

1.相对路径

```
[java]import “./model” //当前文件同一目录的model目录，但是不建议这种方式来import[/java]
```

2.绝对路径

import “shorturl/model” //加载gopath/src/shorturl/model模块

上面展示了一些import常用的几种方式，但是还有一些特殊的import，让很多新手很费解，下面我们来一一讲解一下到底是怎么一回事

**1.点操作**

我们有时候会看到如下的方式导入包

```
[java]import(
. "fmt"
)[/java]
```

这个点操作的含义就是这个包导入之后在你调用这个包的函数时，你可以省略前缀的包名，也就是前面你调用的fmt.Println(“hello world”)可以省略的写成Println(“hello world”)

**2.别名操作**

别名操作顾名思义我们可以把包命名成另一个我们用起来容易记忆的名字

```
[java]import(
f "fmt"
)[/java]
```

别名操作的话调用包函数时前缀变成了我们的前缀，即f.Println(“hello world”)

**3._操作**

这个操作经常是让很多人费解的一个操作符，请看下面这个import

```
[java]import&amp;amp;nbsp;(
"database/sql"
_ "github.com/ziutek/mymysql/godrv"
)[/java]
```

_操作其实是引入该包，而不直接使用包里面的函数，而是调用了该包里面的init函数，要理解这个问题，需要看下面这个图，理解包是怎么按照顺序加载的：

程序的初始化和执行都起始于main包。如果main包还导入了其它的包，那么就会在编译时将它们依次导入。有时一个包会被多个包同时导入，那么它只会被导入一次（例如很多包可能都会用到fmt包，但它只会被导入一次，因为没有必要导入多次）。当一个包被导入时，如果该包还导入了其它的包，那么会先将其它包导入进来，然后再对这些包中的包级常量和变量进行初始化，接着执行init函数（如果有的话），依次类推。等所有被导入的包都加载完毕了，就会开始对main包中的包级常量和变量进行初始化，然后执行main包中的init函数（如果存在的话），最后执行main函数。下图详细地解释了整个执行过程：

[![golang 2.3.init](/wp-content/uploads/2013/10/2.3.init_.png)](/wp-content/uploads/2013/10/2.3.init_.png)


通过上面的介绍我们了解了import的时候其实是执行了该包里面的init函数，初始化了里面的变量，_操作只是说该包引入了，我只初始化里面的init函数和一些变量，但是往往这些init函数里面是注册自己包里面的引擎，让外部可以方便的使用，就很多实现database/sql的引起，在init函数里面都是调用了sql.Register(name string, driver driver.Driver)注册自己，然后外部就可以使用了。

这样我们就介绍完了全部import的情况，希望对你理解Go的import有一定的帮助。