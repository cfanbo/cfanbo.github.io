---
title: python中的@staticmethod和@classmethod区别(整理)
author: admin
type: post
date: 2019-01-12T03:44:08+00:00
url: /archives/18705
categories:
 - 程序开发
tags:
 - python

---
**问题**：Python中 `@staticmethod` 和 `@classmethod` 两种装饰器装饰的函数有什么不同？
**原地址**： [http://stackoverflow.com/questions/136097/what-is-the-difference-between-staticmethod-and-classmethod-in-python](http://stackoverflow.com/questions/136097/what-is-the-difference-between-staticmethod-and-classmethod-in-python)

* * *

Python其实有3类方法：

 * 静态方法（staticmethod）
 * 类方法（classmethod）
 * 实例方法（instance method）

看一下下面的示例代码：

```
def foo(x):
    print "executing foo(%s)" %(x)

class A(object):
    def foo(self,x):
        print "executing foo(%s,%s)" %(self,x)

    @classmethod
    def class_foo(cls,x):
        print "executing class_foo(%s,%s)" %(cls,x)

    @staticmethod
    def static_foo(x):
        print "executing static_foo(%s)" %x

a = A()
```

在示例代码中，先理解下函数里面的 self 和 cls。这个 self 和 cls 是对类或者实例的绑定，对于一般的函数来说我们可以这么调用`foo(x)`，这个函数就是最常用的，它的工作和任何东西（类、实例）无关。**对于实例方法，我们知道在类里每次定义方法的时候都需要绑定这个实例，就是**` **foo(self,x)** `，为什么要这么做呢？因为实例方法的调用离不开实例，我们需要把实例自己传给函数，调用的时候是这样的`a.foo(x)`（其实是`foo(a,x)`）。**类方法一样，只不过它传递的是类而不是实例， 如**` **A.class_foo(x)** `。注意这里的 self 和 cls 可以替换别的参数，但是python的约定是这两个，尽量不要更改。

对于静态方法其实和普通的方法一样，不需要对谁进行绑定，唯一的区别是调用时候需要使用`a.static_foo(x)`或`A.static_foo()`来调用。

 \

 实例方法

 类方法

 静态方法

 a = A()

 a.foo(x)

 a.class_foo(x)

 a.static_foo(x)

 A

 不可用

 A.class_foo(x)

 A.static_foo(x)


那么对于类方法和静态方法都可以不实例对象来调用，那两者的区别又有哪些呢？

**从它们的使用上来看**

 * @staticmethod 不需要表示自身对象的self和自身类的cls参数，就跟使用函数一样。
 * @classmethod 也不需要self参数，但第一个参数需要是表示自身类的cls参数。
 *

如果在@staticmethod中要调用到这个类的一些属性方法，只能直接 **类名.属性名**或 **类名.方法名**。

而@classmethod因为持有 cls 参数，可以来调用类的属性，类的方法，实例化对象等，避免硬编码。

代码:

```
class A(object):
    bar = 1
    def foo(self):
        print 'foo'

    @staticmethod
    def static_foo():
        print 'static_foo'
        print A.bar

    @classmethod
    def class_foo(cls):
        print 'class_foo'
        print cls.bar
        cls().foo()

A.static_foo()
A.class_foo()
```

```
输出
static_foo
1

class_foo
1
foo
```

摘自：
[https://www.cnblogs.com/taceywong/p/5813166.html](https://www.cnblogs.com/taceywong/p/5813166.html) [https://blog.csdn.net/handsomekang/article/details/9615239](https://blog.csdn.net/handsomekang/article/details/9615239)