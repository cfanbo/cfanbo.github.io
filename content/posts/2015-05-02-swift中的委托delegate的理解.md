---
title: swift中的委托delegate的理解
author: admin
type: post
date: 2015-05-01T16:53:33+00:00
url: /archives/15670
categories:
 - 程序开发
tags:
 - swift

---
对象.delegate=self是啥意思
委托的意思就是将自己的任务交给其他人去做!

对象.delegate=self的意思就是对象的任务交给self去做 对象！=self

假如你有对象A 对象B

A是B的成员变量

class B {

member A

}

在B中写这么一句“**A.delegate=self**”

就是将对象A的任务交给self(这里是B)去完成(默认情况下是由A来完成还是？？？，通过在class B中重写class A 中的一些与对象相关的方法函数来实现。)

**其实还有两方面的理解：**
1.委托是继承的一种实现。比如A委托 给B , B实现了A中的方法。有点类似B继承了A。
2.委托方法能够读取被委托对象的属性和方法，这点可以部分解答了你问的“委托必要性”。

比如A委托 给B，在B中实现的委托方法就可以像A中的其他方法一样访问B中的属性。

官方文档： [http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/21_Protocols.html#delegation](http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/21_Protocols.html#delegation)

委托是一种设计模式(_译者注: 想起了那年 UITableViewDelegate 中的奔跑，那是我逝去的Objective-C。。。_)，它允许`类`或`结构体`将一些需要它们负责的功能`交由(委托)`给其他的类型的实例。

委托模式的实现很简单: 定义`协议`来`封装`那些需要被委托的`函数和方法`， 使其`遵循者`拥有这些被委托的`函数和方法`。

委托模式可以用来响应特定的动作或接收外部数据源提供的数据，而无需要知道外部数据源的所属类型(_译者注:只要求外部数据源`遵循`某协议_)。

```lang-swift
<span class="hljs-keyword">let</span> tracker = <span class="hljs-type">DiceGameTracker</span>()
<span class="hljs-keyword">let</span> game = <span class="hljs-type">SnakesAndLadders</span>()
game.delegate = tracker //委托给tracker对象处理
game.play()
```