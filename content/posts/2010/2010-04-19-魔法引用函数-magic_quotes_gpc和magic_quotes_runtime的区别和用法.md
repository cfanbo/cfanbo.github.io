---
title: 魔法引用函数 magic_quotes_gpc和magic_quotes_runtime的区别和用法
author: admin
type: post
date: 2010-04-19T06:13:45+00:00
url: /archives/3444
IM_contentdowned:
 - 1
categories:
 - 程序开发
 - 服务器
tags:
 - php

---
PHP基础002: 魔法引用函数magic\_quotes\_gpc和magic\_quotes\_runtime的区别和用法

PHP提供两个方便我们引用数据的魔法引用函数magic\_quotes\_gpc和magic\_quotes\_runtime，这两个函数如果在 php.ini设置为ON的时候，就会为我们引用的数据碰到单引号’和双引号”以及反斜线 \ 是自动加上反斜线，帮我们自动转译符号，确保数据操作的正确运行，可是我们在php不同的版本或者不同的服务器配置下，有的 magic\_quotes\_gpc和magic\_quotes\_runtime设置为on，有的又是off，所以我们写的程序必须符合on和off两种情 况。那么magic\_quotes\_gpc和magic\_quotes\_runtime两个函数有什么区别呢？看下面的说明：

**magic\_quotes\_gpc**
作用范围是：ＷＥＢ客户服务端；
作用时间：请求开始是，例如当脚本运行时．

**magic\_quotes\_runtime**
作用范围：从文件中读取的数据或执行exec()的结果或是从ＳＱＬ查询中得到的；
作用时间：每次当脚本访问运行状态中产生的数据．
所以

magic\_quotes\_gpc的设定值将会影响通过Get/Post/Cookies获得的数据
magic\_quotes\_runtime的设定值将会影响从文件中读取的数据或从数据库查询得到的数据

例子说明：

```
$data1 = $_POST[‘aaa’];
$data2 = implode(file(‘1.txt’));
if(get_magic_quotes_gpc()){
//把数据$data1直接写入数据库 (自动转译)
}else{
$data1 = addslashes($data1);
//把数据$data1写入数据库，用函数(addslashes()转译)
}

if(get_magic_quotes_runtime()){
//把数据$data2直接写入数据库(自动转译)

//从数据库读出的数据要经过一次stripslashes()之后输出stripslashes()的作用是去掉:\ ，和addslashes()作用相反
}else{
$data2 = addslashes($data2);
//把数据$data2写入数据库

//从数据库读出的数据直接输出
}
```



最关键的区别是就是上面提到的2点:他们针对的处理对象不同
**magic\_quotes\_gpc的设定值将会影响通过Get/Post/Cookies获得的数据
magic\_quotes\_runtime的设定值将会影响从文件中读取的数据或从数据库查询得到的数据**

在这里顺便在提几个想关联的函数：

set\_magic\_quotes_runtime():
设置magic\_quotes\_runtime值. 0=关闭.1=打开.默认状态是关闭的.可以通过 echo phpinfo(); 查看magic\_quotes\_runtime

get\_magic\_quotes_gpc():
查看magic\_quotes\_gpc值.0=关闭.1=打开.

get\_magic\_quotes_runtime():
查看magic\_quotes\_runtime值。0=关闭.1=打开.

注意的是没有 set\_magic\_quotes\_gpc()这个函数,就是不能在程序里面设置magic\_quotes_gpc的值。