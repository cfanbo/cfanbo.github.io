---
title: Golang中关于defer语句理解的一道题
author: admin
type: post
date: 2020-01-18T04:44:40+00:00
url: /archives/19746
categories:
 - 程序开发
tags:
 - defer
 - golang

---
## 示例 

我们先看一下源代码

```
package main

import "fmt"

func f(n int) (r int) {
	defer func() {
		r += n
		recover()
	}()

	var fc func()
	defer fc()
	fc = func() {
		r += 2
	}

	return n + 1
}

func main() {
	fmt.Println(f(3))
}
```

大家感觉着打印的值是多少呢？5、9还是7？执行完以后发现是7。好像与多数理解的有些出入，为什么是7，而不是9呢。下面我们来分析一下。

## 问题分析 

对于defer执行的顺序是FIFO这一点都很清楚，我们只需要看搞懂f()函数的执行顺序就行了。

**执行顺序为：**

 1. 注册第1个defer 函数, 这里为匿名函数，函数体为 “func() { r += n recover() }()”，内部对应一个函数指针。这里延时函数所有相关的操作一步完成。
 2. 注册第2个defer函数，函数名为fc()，无函数体, 函数指针为**nil**（也有可能指针不会空，但指针指向的内容非函数体类型）。由于只是注册操作还未执行，所以并不会产生错误，继续执行。
 3. 对上面声明的函数进行函数体定义
 4. 执行return 语句
 5. 处理defer语句，根据FIFO原则，首先执行第二个函数fc()，发现函数指针为nil，此时会抛出一个恐慌，并继续操作。
 6. 执行第一个defer函数，对r值进行操作，同时处理恐慌。由于是最后一个defer语句，所以直接将r的值真正返回

可以看到上面第2、3步骤，是先注册的defer函数(函数不存在，所以指针为nil)，再进行的函数体定义，导致第二个defer延时函数执行时产生恐慌，后面对函数体的单独定义没有任何意义，大家可以将此函数删除再次运行会发生没有任何问题，直到第一个defer函数对此处理并返回r值结束。

如果打印恐慌错误信息的话，会输出“runtime error: invalid memory address or nil pointer dereference”。

如果我们将 defer fc()函数函数体定义的下方，则完全不会产生恐慌，此时两个defer都会正常执行，最后的结果为9。

修正后的代码

```
package main

import "fmt"

func f(n int) (r int) {
	defer func() {
		r += n
		// recover()
		if err := recover(); err != nil {
			fmt.Println(err)
		}
	}()

	var fc func()
	// defer fc()
	fc = func() {
		r += 2
	}
	defer fc()

	return n + 1
}

func main() {
	fmt.Println(f(3))
}
```

## 总结 

 * defer延时函数最好使用匿名函数来处理，越简单越好
 * defer语句只执行的时候才会产生恐慌，定义时不会产生。
 * 另外如果在注册defer函数的时候，存在非固定的值，则需要先计算出来值，再进行延时函数注册,如 defer sum(1, sum(10, 20))，自己动手试一下值是多少。