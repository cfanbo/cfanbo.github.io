---
title: Golang中struct结构体的的值方法和指针方法
author: admin
type: post
date: 2018-10-20T07:38:53+00:00
url: /archives/18554
categories:
 - 程序开发
tags:
 - golang

---
推荐：[Go的方法集详解（360云计算）][1]

平时我们在写struct的时候，经常会用到一些方法，有些方法是我们熟悉的普通方法，在golang中我们称之为值方法，而另一种则是指针方法。

```
type Person struct {
    Firstname string
    Lastname string
    Age uint8
}
// 值方法
func (p Person) show() {
    fmt.Println(p.Firstname)
}
// 指针方法
func (p *Person) show2() {
    fmt.Println(p.Firstname)
}
```



可以看到所谓的值方法与指针方法在编写的时候，只是有无*****号的区别，这个*就是指针的意思。

那么用法又有何不同呢？

```
// 值方法
func (p Person) setFirstName(name string) {
    p.Firstname = name
}
// 指针方法
func (p *Person) setFirstName2(name string) {
    p.Firstname = name
}
func main() {
    p := Person{"sun", "xingfang", 30}
    //不一致的情况
    p.show() // sun 修改前
    p.setFirstName("tom")   // 值方法
    p.show() // sun, 未变化
    p.show() // sun 修改前
    p.setFirstName2("tom")  // 指针方法
    p.show() // tom 修改后的tom
}
```



通过上面的输出我们可以看到，当调用值方法setFirstName后，输出的还是原来的值sun，而调用指针方法 setFirstNam2后，则输出的是新值。主要原因就是值方法**func (p Person)**在传递总结体的时候，用的只是原来**结构体的一个副本**，做的任何修改也只是对副本的修改，而打印的还是原来的结构体，两者互不影响。而指针方法，传递的则是**指向结构体指针的值副本**，指针值一样(X012242R424)，指定的都是底层的数据结构，所以才会出现这种情况。

**总结：**

 1. 值方法的接收者是该方法所属的那个类型值的一个副本。我们在该方法内对该副本的修1改一般都不会体现在原值上，除非这个类型本身是某个引用类型（比如切片或字典）的别名类型。 而指针方法的接收者，是该方法所属的那个基本类型值的指针值的一个副本。我们在这样的方法内对该副本指向的值进行修改，却一定会体现在原值上。
 2. 一个自定义数据类型的方法集合中仅会包含它的所有值方法，而该类型的指针类型的方法集合却囊括了前者的所有方法，包括所有**值方法**和**所有指针方法**。

 严格来讲，我们在这样的基本类型的值上只能调用到它的值方法。但是，Go 语言会适时地为我们进行自动地转译，使得我们在这样的值上也能调用到它的指针方法。本示例中的 p.show() 在调用的时候会自动转换成 show2() 这种指针方式，可以试着将例子中的 show() 改成 show2() 输出结果是一样的)比如，在Cat类型的变量cat之上，之所以我们可以通过cat.SetName(“monster”)修改猫的名字，是因为 Go 语言把它自动转译为了(&cat).SetName(“monster”)，即：先取cat的指针值，然后在该指针值上调用SetName方法。以上是由“郝林”老师在“Go语言核心36讲”专栏中总结。
 3. 两种写法在使用接口的时候也会有所不同。

[1]: https://mp.weixin.qq.com/s/msXzSfrDAHNPFjMtJ_i0cw