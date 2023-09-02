---
title: require和include基本的区别
author: admin
type: post
date: 2008-11-13T01:48:41+00:00
excerpt: |
 |
 require() 和 include() 除了怎样处理失败之外在各方面都完全一样。include() 产生一个警告而 require() 则导致一个致命错误。换句话说，如果你想在丢失文件时停止处理页面，那就别犹豫了，用 require() 吧。include() 就不是这样，脚本会继续运行。同时也要确认设置了合适的include_path。

 就是说再解析程序时即读取require的文件，而不是解析后，

 如果不能读取到被require的文件，就不能进行下一步动作。
 所以，不被正确包含就会导致程序的文件，用require比较好。
url: /archives/610
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php

---
手册里是这么解释的：

require()   和   include()   除了怎样处理失败之外在各方面都完全一样。include()   产生一个警告而   require()   则导致一个致命错误。换句话说，如果你想在丢失文件时停止处理页面，那就别犹豫了，用   require()   吧。include()   就不是这样，脚本会继续运行。同时也要确认设置了合适的include_path。

就是说再解析程序时即读取require的文件，而不是解析后，

如果不能读取到被require的文件，就不能进行下一步动作。
所以，不被正确包含就会导致程序的文件，用require比较好。

可能效率上也略微高点。
—————————————————————

require()   无论如何都会包含文件，而   include()   可以有选择地包含：

**a.php   一定会被包含，而   b.php   一定不会被包含。**

在PHP中include和require到底有什么区别呢？看这里的例子就知道了

:include.php3的运行结果是：
　　这是inc1.inc文件中的一个变量的值！
　　这是inc2.inc文件中的一个变量的值！
　　inc1.inc文件中的$int变量值为1！

require.php3的运行结果是：
　　这是inc1.inc文件中的一个变量的值！
　　inc1.inc文件中的$int变量值为2！

你可以看到在require.php3中$int变为了2，也就是说inc1.inc中的语句被执行了2次，这样看来在循环中require语句只被解释一次，而且会把require语句所在的位置用require的文件内容替代并运行，而在循环中include语句每次都会被解释运行。

[sonymusic]补充道：
require是只执行一次的，不，这么说不恰当。应当说，**require是先替代，将指定文件的内容代进来，再运行，**所以它不知道你设置了一FOR循环。**而include语句，是什么时候执行到了，什么把指定文件的内容代进来，**继续执行。

include.php3**


　　”;
　　?>


**
require.php3：**


　　”;
　　?>


**
inc1.inc：**
　　”;
　　if(isset($int)){
　　    $int++;
　　}
　　else{
　　    $int = 1;
　　}
　　?>
**
inc2.inc：**
　　”;
　　?>
**