---
title: nodejs中exports与module.exports的区别
author: admin
type: post
date: 2016-03-10T09:18:53+00:00
url: /archives/16727
categories:
 - 程序开发
tags:
 - nodejs

---
对于两者的理解只要记住一句话：“**exports就是module.exports****的引用**”即可。

推荐易理解的文档： [http://cnodejs.org/topic/5231a630101e574521e45ef8](http://cnodejs.org/topic/5231a630101e574521e45ef8)

原文：

你肯定非常熟悉nodejs模块中的**exports**对象，你可以用它创建你的模块。例如：（假设这是rocker.js文件）

```
exports.name = function() {
    console.log('My name is Lemmy Kilmister');
};
```

在另一个文件中你这样引用

```
var rocker = require('./rocker.js');
rocker.name(); // 'My name is Lemmy Kilmister'
```

那到底**Module.exports**是什么呢？它是否合法呢？

其实，**`Module.exports`**才是真正的接口，**exports**只不过是它的一个辅助工具。　最终返回给调用的是**`Module.exports`**而不是**exports。**

所有的**exports**收集到的属性和方法，都赋值给了**`Module.exports`**。当然，这有个前提，就是**`Module.exports`**`本身不具备任何属性和方法``。如果，`**`Module.exports`**`已经具备一些属性和方法，那么exports收集来的信息将被忽略。```

修改rocker.js如下：

```
module.exports = 'ROCK IT!';
exports.name = function() {
    console.log('My name is Lemmy Kilmister');
};
```

再次引用执行rocker.js

```
var rocker = require('./rocker.js');
rocker.name(); // TypeError: Object ROCK IT! has no method 'name'
```

发现报错：对象“ROCK IT!”没有name方法（因为第二行的exports是对第一行的module.exports的引用，第二句exports.name则表示 module.exports.name，它想给声明一个name方法，但是第一行module.exports = ‘ROCK IT!’已经进行了赋值操作，值为一个字符串常量，所以最后提示错误 “对象ROCK IT! 没有name方法”）。

rocker模块忽略了exports收集的name方法，返回了一个字符串“ROCK IT!”。由此可知，你的模块并不一定非得返回“实例化对象”。你的模块可以是任何合法的javascript对象–boolean, number, date, JSON, string, function, array等等。

你的模块可以是任何你设置给它的东西。如果你没有显式的给**`Module.exports`**``设置任何属性和方法，那么你的模块就是exports设置给**`Module.exports的`**`属性（这里再次说明了对exports的操作实际上可视为对Module.exports的操作，因为exports是对Module.exports的引用）。```

下面例子中，你的模块是一个类：

```
module.exports = function(name, age) {
 this.name = name;
 this.age = age;
 this.about = function() {
     console.log(this.name +' is '+ this.age +' years old');
 };
};
```

可以这样应用它：


```
var Rocker = require('./rocker.js');
var r = new Rocker('Ozzy', 62);
r.about(); // Ozzy is 62 years old
```

下面例子中，你的模块是一个数组：

```
module.exports = ['Lemmy Kilmister', 'Ozzy Osbourne', 'Ronnie James Dio', 'Steven Tyler', 'Mick Jagger'];
```

可以这样应用它：

```
var rocker = require('./rocker.js');
console.log('Rockin in heaven: ' + rocker[2]); //Rockin in heaven: Ronnie James Dio
```

现在你明白了，如果你想你的模块是一个特定的类型就用**`Module.exports`**``。如果你想你的模块是一个典型的“实例化对象”就用**exports**。

给Module.exports添加属性类似于给exports添加属性(因为exports是对Module.exports的引用)。例如：

```
module.exports.name = function() {
    console.log('My name is Lemmy Kilmister');
};
```

同样，exports是这样的

```
exports.name = function() {
    console.log('My name is Lemmy Kilmister');
};
```

请注意，这两种结果并不相同。前面已经提到module.exports是真正的接口，exports只不过是它的辅助工具。推荐使用exports导出，除非你打算从原来的“实例化对象”改变成一个类型。