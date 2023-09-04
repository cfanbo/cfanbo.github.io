---
title: jQuery.extend和jQuery.fn.extend的区别-转
author: admin
type: post
date: 2016-02-23T03:38:20+00:00
url: /archives/16700
categories:
 - 前端设计
tags:
 - js

---
jQuery.extend和jQuery.fn.extend的区别，其实从这两个办法本身也就可以看出来。很多地方说的也不详细。这里详细说说之间的区别.

1. 我们先把jQuery看成了一个类，这样好理解一些。


jQuery.extend()，是扩展的jQuery这个类。


假设我们把jQuery这个类看成是人类，能吃饭能喝水能跑能跳，现在我们用jQuery.extend这个方法给这个类拓展一个能唱歌的技能。这样的话，不论是男人，女人，xx人…..等能继承这个技能（方法）了。


可以如下图这样写着：

[![jquery-1](https://blogstatic.haohtml.com//uploads/2023/09/jquery-1.png)](http://blog.haohtml.com/wp-content/uploads/2016/02/jquery-1.png)

2. 然后：$.liu();这样就能打印出来”liu“这个字符串


代码在下面：

[![jquery-2](https://blogstatic.haohtml.com//uploads/2023/09/jquery-2.png)](http://blog.haohtml.com/wp-content/uploads/2016/02/jquery-2.png) 3. 这说明啥啊，这说明.liu()变成了jQuery这个类本身的方法（object）嘛。他现在能”唱歌“了。但是吧，这个能力啊，只有代表全人类的 jQuery 这个类本身，才能用啊。你个人想用，你张三李四王五麻六，你个小草民能代表全人类嘛？


所以啊，这个扩展也就是所谓的静态方法。只跟这个 类 本身有关。跟你具体的实例化对象是没关系滴。


我们再看看jQuery.fn.extend()这个方法。


从字面理解嘛，这个拓展的是jQuery.fn的方法。


jQuery.fn是啥玩意呢？


源码如下


![jquery-3](https://blogstatic.haohtml.com//uploads/2023/09/jquery-3.png)

4. 哦，原来jQuery.fn=jQuery.prototype，就是原型啊。


那就一目了然了，jQuery.fn.extend拓展的是jQuery对象（原型的）的方法啊！


对象是啥？就是类的实例化嘛，例如


$(“#abc”)


这个玩意就是一个实例化的jQuery对象嘛。


那就是说，jQuery.fn.extend拓展的方法，你得用在jQuery对象上面才行啊！他得是张三李四王五痳六这些实例化的对象才能用啊。


说白了就是得这么用（假设xyz()是拓展的方法）：


$(‘selector’).xyz();


你要是这么用$.xyz()；是会出错误滴。


代码看下面图片：

[![jquery-4](https://blogstatic.haohtml.com//uploads/2023/09/jquery-4.png)](http://blog.haohtml.com/wp-content/uploads/2016/02/jquery-4.png)

转： [http://jingyan.baidu.com/article/fec4bce259ef67f2608d8b10.html](http://jingyan.baidu.com/article/fec4bce259ef67f2608d8b10.html)