---
title: link和@import引用css文件方法的区别
author: admin
type: post
date: 2008-11-12T07:28:44+00:00
url: /archives/549
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - css

---

元素所参考的样式用户可以自由的选择加以改变，而导入的样式表单就自动的与剩下的样式表融合在一起了


CSS与HTML文档结合的4中方法：

1 使用元素链接到外部的样式文件

2 在元素中使用”style”元素来指定

3 使用CSS “@import”标记来导入样式表单

4 在内部的元素中使用”style”属性来定义样式


一个例子:

css demo

第一种是直接在html页面上进行css书写，而第二种和第三种是采用外部引用样式单独提取文件。


问题1.到底link和@import有什么区别？

我们先来看看他们的定义

link元素

HTML和XHTML都有一个结构，它使网页作者可以增加于HTML文档相关的额外信息。这些额外资源可以是样式化信息（CSS）、导航助手、属于另 外形式的信息（RSS）、联系信息等等。


@import

指定导入的外部样式表及目标设备类型。

其实link和@import的最根本区别就是，**link** 是一个 **html** 的一个标签 ，而**@import** 是 **css** 的一个标签 ,

link除了调用css外还可以有其他作用譬如声明页面链接属性，声明目录，rss等等，而@import就只能

调用css。如果单独从外部引用css来说，他们的作用是基本一样，只不过上面的老大不一样而已。：）


问题2.link合import到底那个更好？

上面说了因为上面的老大不一样，所以在使用上就会有一些细节的区别，不能说总体谁好谁坏，

只能说具体情况具体分析。

1）我要用javascript进行样式选择；

这个时候就要用link,因为link是html元素，可用javascript去控制dom元素最后达到改变样式的效果。

看下列代码


这是一段很经典的改变页面风格的代码，因为我们今天主要讲的是link和import，所以我这里只列出了引用css部分。

我们先来看看link里面个个属性都是表达了什么意思：

[1]rel:用来声明链接对象的作用或者类型。

譬如上面的的代码：”stylesheet”表示链接一个默认的css,而”alternate stylesheet”折表示备选的css

[2]href:这个就不用我说了吧，引用css的文件路径。

[3]tyle:文件类型

[4]media:应用的设备，”screen”是说明应用在屏幕上。

[5]title:是css的名称。

这段代码中一共有5个css,第一个是基本样式，而其他四个是风格样式,利用javascript去控制默认显示的样式title就ok了。


2）我要在应用打印样式；

打印样式顾名思义就是打印页面时候的样式。

这个样式在普通浏览下是没有效果的，只有在打印的时候生效。

如果要为页面单独引用打印样式的话，link和@import都可以的。


link代码


@import代码


另外对于css来说还有一种方式@media：


@media print {

@import “print.css”

}

用@media先制定设备为 print,然后再用@impor链接


3）我要引用多个样式；

如果要在一个页面上引用多个样式组合产生效果的话，永link和@import也是都可以的。


link代码


@import代码


不过个人觉得，用@import引用多文件的时候更加清晰一些

另外对于多样式还有一种link于@import的组合用法。

先用link引用一个css文件


然后在这个css文件里面再引用。


这样做的好处是，如果你一个站点所有页面引用的样式都是一样的，

而有又多个css，如果你每个页面都加4,5个一样的css样式，却是浪费代码和精力，

所以莫不如这样做，所有一个页面都引用一个css，然后一个css在引用多个css，方便

管理和维护。


加载css link与@import的区别：其实 link 与 @import 在显示效果上还是有很大区别的，基本上来看 link 的加在会在页面显示之前全部加在完全，而 @import 会是读取完文件之后加在，所以如果网速很好或很快的情况下，会出现先开始无css定义，而后加载css定义。@import加载页面时开始的瞬间会有闪烁（无样式表的页面），然后才恢复正常（加载样式后的页面），Link没有这个问题。


他们从方法上是一样的，只是在浏览器识别上有点差距，link在支持CSS的浏览器上都支持而@import只在5.0一行的版本有效，而且还能用于浏览器过滤也就是hack的使用，兼容一些老版本的浏览器。所以最好还是使用link通用型更强，但是对于高版本的浏览器，也就是现在的浏览器来说，其实都一样，这是个没有太大意义的区分 。


@import最优写法：@import的写法一般有下列几种：

@import ‘style.css’ //Windows IE4/ NS4, Mac OS X IE5, Macintosh IE4/IE5/NS4不识别

@import “style.css” //Windows IE4/ NS4, Macintosh IE4/NS4不识别

@import url(style.css) //Windows NS4, Macintosh NS4不识别

@import url(‘style.css’) //Windows NS4, Mac OS X IE5, Macintosh IE4/IE5/NS4不识别

@import url(“style.css”) //Windows NS4, Macintosh NS4不识别

由上分析知道，@import url(style.css) 和@import url(“style.css”)是最优的选择，兼容的浏览器也是最多了。而从字节优化的角度来看@import url(style.css)无愧于最值得推荐的写法。


CSS代码格式可以缩减样式表文件的大小

在编写CSS样式表的时候，为了能够方便以后阅读样式定义代码，我们会将每一条代码写在一行上。但是我发现这样写似乎并不好，因为CSS代码毕竟不像程序代码有很强的逻辑性，它主要以名称和值的对应方式写的。所以写在同一行上不会特别影响阅读。反而会减少样式表文件的尺寸，因为减少了很多换行符和间隔符。我测试了一下发现文件的尺寸可以减少12%左右。如果样式表文件比较大或者文件比较多的时候就会明显减少客户端的下载量，提高了网页的打开速度。

注意样式名称的冒号和后面的值之间不要写空格，只是在两个样式之间用空格分割。

具体格式如下：

程序代码：

div {margin:20px; padding:10px; background-color:#F00;}