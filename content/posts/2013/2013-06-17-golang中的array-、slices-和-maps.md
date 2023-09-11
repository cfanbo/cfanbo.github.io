---
title: 'golang中的Array 、Slices 和 Maps'
author: admin
type: post
date: 2013-06-17T06:56:19+00:00
url: /archives/13995
categories:
 - 程序开发
tags:
 - golang

---
**注意`slice`和数组在声明时的区别：**声明数组时，方括号内写明了数组的长度或使用`...`自动计算长度，而声明`slice`时，方括号内没有任何字符。

```
arr1 := [10]int{1,2,3,4} //数组,长度为10,只有4个元素指定,其它的元素值默认为0
arr2 := [...]string{"a","b","c"} //数组,长度自适应,这里长度为3
s1 := []int{1,2,3,4} //slice,目前长度为4,可能通过append来动态添加元素个数
```

示例:

```
package main

import (
 "fmt"
)

func main() {

//array example
 arr := [10]int{1, 2, 3} //array 指定前三个值,其它值使用默认类型值0
 fmt.Println(len(arr))
 fmt.Println(arr)
 //a1 := append(arr, 4, 5) //数组不支持append,只有slice才支持append
 //fmt.Println(a1)

//slice example
 slice := []string{"a", "b", "c"} // slice
 s1 := append(slice, "d")
 fmt.Println(s1)

slice2 := []string{"x", "y", "x"}
 len1 := len(slice2)
 cap1 := cap(slice2)
 fmt.Println(slice2) // [x y x]
 fmt.Println("len:", len1, "cap:", cap1) //len: 3 cap: 3

s2 := append(slice, slice2...)
 len2 := len(slice2)
 cap2 := cap(s2)
 fmt.Println(s2) //[a b c x y x]
 fmt.Println("len:", len2, "cap:", cap2) //len: 3 cap: 6

}
```

===========================================

数组定义方式 “[n]” 有点古怪。长度下标 n 必须是编译期正整数常量 (或常量表达式)。
长度是类型的组成部分，也就是说 “[10]int” 和 “[20]int” 是完全不同的两种数组类型。

```
var a [4]int;  // 所有元素⾃动被初始化为 0
a[1] = 100;
for i := 0; i &amp;amp;lt; len(a); i++ {
	println(a[i])
}
```

可以用复合语句直接初始化。

```

a := [10]int{ 1, 2, 3, 4 }  // 未提供初始化值的元素为默认值 0
b := [...]int{ 1, 2 }  // 由初始化列表决定数组长度，不能省略 "..." ，否则就成 slice 了。
c := [10]int{ 2:1, 5:100 }  // 按序号初始化元素

```

可以像这样声明一个数组：**var a [3]int**，如果不使用零来初始化它，则用复合声
明：**a := [3]int{1, 2, 3}**也可以简写为**a := […]int{1, 2, 3}**，Go会自动统计元素的
个数。注意，所有项目必须都指定。因此，如果你使用多维数组，有一些内容你必须录入：
a := \[2\]\[2\]int{ [2]int{1,2}, [2]int{3,4} }
类似于：
a := \[2\]\[2\]int{ […]int{1,2}, […]int{3,4} }当声明一个array时，你必须在方括号内输入些内容，数字或者三个点(…)。

从release.2010-10-27这个语法使用更加简单了。来自于发布记录：array、slice和map的复合声明变得更加简单。使用复合声明的array、slice和map，元素复合声明的类型与外部一直，则可以省略。

这表示上面的例子可以修改为：a := [2][2]int{ {1,2}, {3,4} }

给定一个array或者其他slice，一个新slice通过a[I:J]的方式创建。这会创建一个新
的slice，指向a，从序号I开始，结束在序号J之前。长度为J – I。
// array[n:m] 从array 创建了一个slice，具有元素n to m-1
a := […]int{1, 2, 3, 4, 5}   //0
s1 := a[2:4]  // 1
s2 := a[1:5]  // 2
s3 := a[:]  // 3
s4 := a[:4]  // 4
s5 := s2[:]  // 5

0 定义一个5个元素的array，序号从0到4；
1 从序号2至3创建slice，它包含元素3, 4；
2 从序号1至4创建，它包含元素2, 3, 4, 5；
3 用array中的所有元素创建slice，这是a[0:len(a)]的简化写法；
4 从序号0至3创建，这是a[0:4]的简化写法，得到1, 2, 3, 4；
5 从slices2创建slice，注意s5仍然指向array a。

如果其中一个slice中对某个数组元素进行了修改的话,则会影响整体包含这个数组元素的slice

数组指针类型 \*[n]T，指针数组类型 [n]\*T 。

```go

x, y := 1, 2
var p1 *[2]int = [2]int{x, y}
var p2 [2]*int = [2]*int{x, y}
fmt.Printf("%#vn", p1)
fmt.Printf("%#vn", p2)

```

输出:

```

[2]int{1, 2}
[2]*int{(*int)(0x4212f100), (*int)(0x4212f108)}

```

支持 “==”、”!=” 相等操作符 (因为 Go 对象所使用的内存都被初始化为 0，按字节比较是可以的)，
但不支持 “>” 、”<” 等比较操作符。

```
println([1]string{"a"} == [1]string{"a"})
```

数组是值类型，也就是说会拷贝整个数组内存进行值传递。可用 slice 或指针代替。

```

func test(x *[4]int) {
  for i := 0; i < len(x); i++ {
    println(x[i]) // 用指针访问数组语法并没有什么不同，比 C 更直观。
  }
  x[3] = 300
}

func main() {
  x := [4]int{ 2:100, 1:200 }
  test(x)
  println(x[3])
}

```

可以用 new() 创建数组，返回数组指针。

```
func test(a *[10]int) {
	a[2] = 100 // 用指针直接操作没有压力。
}

func main() {
  var a = new([10]int)  // 返回指针。
  test(a)
  fmt.Println(a, len(a))
}
```

输出:

```

[0 0 100 0 0 0 0 0 0 0] 10

```

多维数组和 C 类似，一种数组的数组。

```
func main() {
  var a = [3][2]int{ [...]int{1, 2}, [...]int{3, 4} }
  var b = [3][2]int{ {1, 2}, {3, 4} }
  c := [...][2]int{ {1, 2}, {3, 4}, {5, 6} }   // 第二个维度不能用 "..." 。
  c[1][1] = 100
  fmt.Println(a, "n", b, "n", c, len(c), len(c[0]))
}

```

输出：

```

[[1 2] [3 4] [0 0]]
[[1 2] [3 4] [0 0]]
[[1 2] [3 100] [5 6]] 3 2

```

由于元素类型相同，因此可以使用复合字面值初始化数组成员。

```

type User struct {
Id int
Name string
}
func main() {
a := [...]User {
{ 0, "User0" },
{ 1, "User1" },
}
b := [...]*User {
{ 0, "User0" },
{ 1, "User1" },
}
fmt.Println(a)
fmt.Println(b, b[1])
}

```

输出:

```

[{0 User0} {1 User1}]
[0x42122340 0x42122320] {1 User1}
3.2 Slices

```

**3.2 Slices**

作为变长数组的替代方案，slice 具备更灵活的特征。通过相关属性关联到底层数组的局部或全部。

src/pkg/runtime/runtime.h

```
struct  Slice
{ // must not move anything
byte* array; // actual data
uint32 len; // number of elements
uint32 cap; // allocated number of elements
};

```

 [http://blog.golang.org/2011/01/go-slices-usage-and-internals. html](http://blog.golang.org/2011/01/go-slices-usage-and-internals. html)
slice 是引⽤类型，默认值为 nil。可以⽤内置函数 len() 获取⻓度，cap() 获取容量。使用索引访问时，不能超出 0 到 len – 1 范围。和值类型 array 不同，引用类型 slice 赋值不会复制底层数组。
用 slice 长时间引和 “超大” 的底层数组，会导致严重的内存浪费。可考虑新建一个小的 slice 对象，然后将所需的数据 copy 过去。
在使方法上和 Python 类似，可惜不支持负数定义逆向索引。

```

func main() {
x := [...]int{ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 }
var s1 []int = x[1:3] // 内容包括 x[1], x[2]
fmt.Println(s1)
s2 := x[4:] // x[4] ~ x[len - 1]
fmt.Println(s2)
s3 := x[:6] // x[0] ~ x[5]
fmt.Println(s3)
s4 := x[:] &amp;amp;nbsp;// x[0] ~ x[len - 1]
fmt.Println(s4)
}
```

输出:

```

[1 2]
[4 5 6 7 8 9]
[0 1 2 3 4 5]
[0 1 2 3 4 5 6 7 8 9]
```

对 slice 的修改就是对底层数组的修改。

```

func main() {
x := [...]int{ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 }
s := x[1:3]
s[0] = 100  // 对 slice 的修改实际上是对原数组的修改，注意序号的差异。
fmt.Println(x)
x[2] = 200 // 对数组修改会影响 slice 。
fmt.Println(s)
}

```

输出:

```

[0 100 2 3 4 5 6 7 8 9]
[100 200]
```

可以直接创建一个 slice 对象 (内部自动创建底层数组)，而不是从数组开始。

```

func test(s []int) {
fmt.Println(s)
s[1] = 100  // 内部总是指向同⼀个数组。
}
func main() {
s := []int{ 0, 1, 2 }
test(s)  // 引用类型，不用担心复制底层数组。

fmt.Println(s)
}

```

输出:

```

[0 1 2]
[0 100 2]
```

不能使用 new()，而应该是 make([]T, len, cap)。因为除了分配内存，还需要设置相关的属性。如
果忽略 cap 参数，则 cap = len 。

```

func main() {
s1 := make([]int, 10)  // 相当于 [10]int{...}[:]
s1[1] = 100
fmt.Println(s1, len(s1), cap(s1))
s2 := make([]int, 5, 10)
s2[4] = 200
fmt.Println(s2, len(s2), cap(s2))
}
```

输出:

```

[0 100 0 0 0 0 0 0 0 0] 10 10
[0 0 0 0 200] 5 10

```

**3.2.1 reslice**
cap 是 slice 非常重要的属性，它表示了 slice 可以容纳的最大元素数量。在初始创建 slice 对象时
cap = array\_length – slice\_start_index ，也就是 slice 在数组上的开始位置到数组结尾的元素
数量。

在 cap 允许的范围内我们可以 reslice，以便操作后续的数组元素。超出 cap 限制不会导致底层数
组重新分配，只会引发 “slice bounds out of range” 错误。

```
func main() {
 a := [...]int{ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 }
 s1 := a[5:7] &amp;amp;nbsp;// [5 6], len = 2, cap = 5
 s1 = s1[0:4] // [5, 6, 7, 8], len = 4, cap = 5
// 注意 reslice 不能超过 cap 的限制。
// 是在 slice 上重新切分，不是 Array ，因此序号是以 slice 为准。
 }
```

**3.2.2 append**

可以用 append() 向 slice 尾部添加新元素，这些元素保存到底层数组。append 并不会影响原 slice的属性，它返回变更后新的 slice 对象。如果超出 cap 限制，则会重新分配底层数组。

```
func main() {
 s1 := make([]int, 3, 6)
 // 添加数据，未超出底层数组容量限制。
 s2 := append(s1, 1, 2, 3)
 // append 不会调整原 slice 属性。
 // s1 == [0 0 0] len:3 cap:6
 fmt.Println(s1, len(s1), cap(s1))
 // 注意 append 是追加，也就是说在 s1 尾部添加。
 // s2 == [0 0 0 1 2 3] len:6 cap:6
 fmt.Println(s2, len(s2), cap(s2))
 // 追加的数据未超出底层数组容量限制。
 // 通过调整 s1 ，我们可以看到依然使⽤的是原数组。
 // s1 == [0 0 0 1 2 3] len:6 cap:6
 s1 = s1[:cap(s1)]
 fmt.Println(s1, len(s1), cap(s1))
 // 再次追加数据&amp;amp;nbsp;(使用了变参)。
 // 原底层数组已经无法容纳新的数据，将重新分配内存，并拷贝原有数据。
 // 我们通过修改数组第一个元素来判断是否指向原数组。

// s3 == [100 0 0 1 2 3 4 5 6] len:9 len:12
 s3 := append(s2, []int{ 4, 5, 6 }...)
 s3[0] = 100
 fmt.Println(s3, len(s3), cap(s3))
 // 而原 slice 对象依然指向旧的底层数组对象，所以彻底和 s3 分道扬镳。
 // s1 == [0 0 0 1 2 3] len:6 cap:6
 // s2 == [0 0 0 1 2 3] len:6 cap:6
 fmt.Println(s1, len(s1), cap(s1))
 fmt.Println(s2, len(s2), cap(s2))
 }
```

输出:

```
[0 0 0] 3 6
[0 0 0 1 2 3] 6 6
[0 0 0 1 2 3] 6 6
[100 0 0 1 2 3 4 5 6] 9 12
[0 0 0 1 2 3] 6 6
[0 0 0 1 2 3] 6 6
```

因为 append 每次会创建新的 slice 对象，因此要优先考虑⽤序号操作。

```

func test1(n int) []int {
datas := make([]int, 0, n)
for i := 0; i &amp;amp;lt; n; i++ {
datas = append(datas, i)
}
return datas
}
func test2(n int) []int {
datas := make([]int, n)
for i := 0; i &amp;amp;lt; n; i++ {
datas[i] = i
}
return datas
}
func main() {
// datas := test1(10000)
datas := test2(10000)
println(len(datas))
}
```

两种写法的性能差异很明显。虽然底层数组相同，但 test2 预先就 “填充” 了全部元素，演变为普通
数组操作。test1 append 每次循环都会创建新的 slice 对象，累加起来的消耗并不小。

**3.2.3 copy**
函数 copy ⽤于在 slice 间复制数据，可以是指向同⼀底层数组的两个 slice。复制元素数量受限于
src 和 dst 的 len 值 (两者的最⼩值)。在同⼀底层数组的不同 slice 间拷贝时，元素位置可以重叠。

```

func main() {
s1 := []int{ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 }
s2 := make([]int, 3, 20)
var n int
n = copy(s2, s1) // n = 3。不同数组上拷贝。s2.len == 3 ，只能拷 3 个元素。
fmt.Println(n, s2, len(s2), cap(s2)) // [0 1 2], len:3, cap:20
s3 := s1[4:6] // s3 == [4 5] 。s3 和 s1 指向同一个底层数组。
n = copy(s3, s1[1:5]) // n = 2。同一数组上拷贝，且存在重叠区域。
fmt.Println(n, s1, s3) // [0 1 2 3 1 2 6 7 8 9] [1 2]
}
```

输出:

```

3 [0 1 2] 3 20
2 [0 1 2 3 1 2 6 7 8 9] [1 2]

```

**3.3 Maps**
引用类型，类似 Python dict ，不保证 Key/Value 存放顺序。Key 必须是支持比较运算符 (== 、!=)的类型。如 number 、string 、pointer、array、struct 、interface (接口实现类型必须支持比较运算符)，不能是 function、map、slice。
map 查找操作比线性搜索快很多，但比起用序号访问 array、slice，大约慢 100x 左右。绝大多数时候，其操作性能要略好于 Python Dict 、C++ Map。

```

func test(d map[string]int) {
d["x"] = 100
}
func main() {
var d = map[string]int{ "a":1, "b":2 };
d2 := map[int]string{ 1:"a", 2:"b" };
test(d)
fmt.Println(d, d2)
d3 := make(map[string]string)
d3["name"] = "Jack"
fmt.Println(d3, len(d3))
}
```

输出:

```

map[a:1 b:2 x:100] map[1:a 2:b]
map[name:Jack] 1

```

使⽤ array/struct key 的例⼦。

```

type User struct {
Name string
}
func main() {
a := [2]int{ 0, 1}
b := [2]int{ 0, 1}
d := map[[2]int]string { a: "ssss" }
fmt.Println(d, d[b])
u := User{ "User1" }
u2 := u
d2 := map[User]string { u: "xxxx" }
fmt.Println(d2, d2[u2])
}
```

输出:

```

map[[0 1]:ssss] ssss
map[{User1}:xxxx] xxxx

```

value 的类型就很自由了，完全可以用匿名结构或者空接口。

```

type User struct {
Name string
}
func main() {
i := 100
d := map[*int]struct{ x, y float64 } { &amp;amp;amp;i: { 1.0, 2.0 } }
fmt.Println(d, d[&amp;amp;amp;i], d[&amp;amp;amp;i].y)
d2 := map[string]interface{} { "a": 1, "b": User{ "user1" } }
fmt.Println(d2, d2["b"].(User).Name)
}
```

输出:

```

map[0x42132018:{1 2}] {1 2} 2
map[a:1 b:{user1}] user1
```

使用 make() 创建 map 时，提供一个合理的初始容量有助于减少后续新增操作的内存分配次数。在需要时，map 会⾃动扩张容量。
常用的判断和删除操作：

```

func main() {
var d = map[string]int{ "a":1, "b":2 };
v, ok := d["b"]  // key b 存在，v = ["b"], ok = true
fmt.Println(v, ok)
v = d["c"]  // key c 不存在，v = 0 (default)
fmt.Println(v)  // 要判断 key 是否存在，建议用 ok idiom 模式。
d["c"] = 3   // 添加或修改
fmt.Println(d)
delete(d, "c")  // 删除。删除不存在的 key，不会引发错误。
fmt.Println(d)
}
```

输出:

```

2 true
0 false
map[a:1 c:3 b:2]
map[a:1 b:2]
```

迭代器用法:
The Go 1 .1 hashmap iteration starts at a random bucket. However, it stores up to 8 key/value pairs in a bucket.
And within a bucket, hashmap iteration always returns the values in the same order .

```go
func main() {
d := map[string]int{ "a":1, "b":2 };
for k, v := range d {  // 获取 key, value
println(k, "=", v)
}
for k := range d {  // 仅获取 key
println(k, "=", d[k])
}
}
```

通过 map[key] 返回的只是一个 “临时值拷贝”，修改其自身状态没有任何意义，只能重新 value 赋值或改用指针修改所引用的内存。

```
type User struct {
Id int
Name string
}
func main() {
users := map[string]User{
"a": User{1, "user1"},
}
fmt.Println(users)
// Error: cannot assign to users["a"].Name
//users["a"].Name = "Jack"
// 重新 value 赋值
u := users["a"]
u.Name = "Jack"
users["a"] = u
fmt.Println(users["a"])
// 改⽤指针类型
users2 := map[string]*User{
"a2": User{2, "user2"},
}
users2["a2"].Name = "Tom"
fmt.Println(users2["a2"])
}
```

输出:

```

map[a:{1 user1}]
{1 Jack}
{2 Tom}
```

可以在 for range 迭代时安全删除和插入新的字典项。

```
func main() {
d := map[string]int { "b":2, "c":3, "e":5 }
for k, v := range d {
println(k, v)
if k == "b" { delete(d, k) }
if k == "c" { d["a"] = 1 }
}
fmt.Println(d)
}
```

输出：

```

c 3
b 2
e 5
map[a:1 c:3 e:5]
```

注：map 作为参数时，直接复制指针。