---
title: golang中实现自定义数据类型struct
author: admin
type: post
date: 2013-04-23T11:00:00+00:00
url: /archives/13766
categories:
 - 程序开发
tags:
 - golang

---

可以参考： [golang中的函数](http://blog.haohtml.com/archives/13556)

**func.go**

```go
package main

import (
"fmt"
)

type stu struct {
Name string //首字母大写，允许其它包直接使用，可以直接使用 stu.Name = 'test' 也可以使用 setName和getName
age int //不允许外面的包使用,可以使用 setAge和getAge方法
}

func main() {

perl := new(stu)
perl.Name = "zhang"

// age
setAge(perl, 30)
age := getAge(perl)

fmt.Printf("%v\n", age)

//name
var name string
perl.setName("sun")
name = perl.getName()

fmt.Printf("%i\n", name)

//print struct
fmt.Printf("%v\n", perl)
}

func setAge(s *stu, age int) {

s.age = age
}

func getAge(s *stu) int {
return s.age
}

```

//========= 另一种写法

```go
func (s *stu) setName(name string) {
s.Name = name
}

func (s *stu) getName() string {
return s.Name
}

```

对于结构体struct的初始化的几种方法，见：http://blog.haohtml.com/archives/14239