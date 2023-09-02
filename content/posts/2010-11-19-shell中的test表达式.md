---
title: shell中的test表达式
author: admin
type: post
date: 2010-11-19T07:59:00+00:00
url: /archives/6727
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - shell
 - test

---
摘自：UNIX Shell编程24学时编程

**10.1.2 使用test**

更常见的情况是，提供给if语句的清单是一个或更多个test命令，它们通过调用test命令而被激活，语法如下：

> test expression

这里，expression为test命令构造的表达式，该表达式的构造使用了特殊选项之一.在计算完表达式的值后，test命令或者返回0(真)或者返回1(假).

可用”[“命令对test命令进行缩写:

> ［ expression ］

这里expression是test命令可以理解的任何有效表达式，该简化格式将是读者可能会踫见的最常用格式.

test可理解的表达式类型分为三类：

> 文件测试．
> 字符串比较.
> 数字比较.

读者将逐步学业习这三类，另外，还会学习到复合表达式.

注意：在使用”[“简写test时，左中括号后面的空格和右括号前面的空格是必需的，如果没有空格，Shell不可能辨别表达式何时开始何时结束．

 选项

 描述

 -b file

 若文件存在且是一个块特殊文件，则为真

 -c file

 若文件存在且是一个字符特殊文件,则为真

 -d file

 若文件存在且是一个目录，则为真

 -e file

 若文件存在则为真

 -f file

 若文件存在且为一个规则文件则为真

 -g file

 若文件存在且设置了SUID位的值，则为真

 -h file

 若文件存在且为一个符号链接，则为真

 -k file

 若文件存在且设置了”sticky”位的值，则为真

 -p file

 若文件存在为已命令管道，则为真

 -r file

 若文件存在且可读，则为真

 -s file

 若文件存在且其大小大于零，则为真

 -w file

 若文件存在且设置了SUID位的值，则为真

 -x file

 若文件存在且可执行，则为真

 -o file

 若文件存在且被有效用户ID所拥有，则为真


**1.文件测试**

文件测试表达式检查是否一个文件满足某种特殊规则．文件测试的通用语法为：

> test option file

或

> [ option file ]

这里optionj 上表给出的一个选项，file是文件或目录名．

下面看一些if语句的实例，这些实例使用了test命令执行文件测试．

考虑如下if语句：

> $if [ -d /home/ranga/bin ]; then PATH = “$ PATH:/home/ranga/bin”; fi

这里，测试了目录/home/ranga/bin是否存在,若存在，将其增加到变量　PATH　上．在Shell初始化脚本　如.profile或.kshrc中常见类似语句．

假设想在$HOME/.bash_aliai文件存在时执行存储在其中的命令，可使用命令：

> $if [ -f $HOME/./bash\_aliai ]; then $HOME/.bash\_aliai; fi

对该命令的一个改进是：增加测试该文件是否有内容这一功能，若有，则执行其中的命令．可使用-s选项代替-f选项改变该命令得到这样的结果．

> if [-s $HOME/.bash\_aliai ];  then $HOME/.bash\_aliai; fi

现在，若文件存在且有一些内容，则执行存储在文件$HOME/.bash_aliai中命令．

**2.字符串比较**

test命令也运行简单的字符串比较，主要存在两种格式：

1) 检查是否字符串为空

2)  检查是否两个字符串相等

字符串不能使用test命令直接与一个表达式进行比较．若这样，则要用case语句代替.

写字符串比较有关的test选项在下表给出:

 选项

 描述

 -z string

 若string长度为0，则为真

 -n string

 若string长度不为0，则为真

 string! = string2

 若两个字符串相等，则为真

 string1 != string2

 若两个字符串不等，则为真


检查是否字符串为空

第一种格式的语法为:

> test option string

或

> [ option string ]

这里option或者是-z或者是-n,string为任何有效的Shell字符串，-z选项(zero的首字母)检查是否字符串长度为0，而-n选项检查是否字符串长度不为0.

例如，如下命令：

> if [ -z “$FRUIT _BASKER” ] ; then
> echo “Your fruit basket is empty”;
> else
> echo “your fruit basket has the following fruit; $FRUIT_BASKET”
> fi

若包含在变量$FRUIT_BASKET的字符串长度为0，则产生字符串:

> Your fruit basket is empty

若要使用-n选项代替-z选项，本例变为：

> if [ -n “$FRUIT_BASKET” ]  ; then
> echo “Your fruit basket has the following fruit:$FRUIT_BASKET”
> else
> echo “Your fruit basket is emtpy”;
> fi

注意在本例中引用了变量＄FRUIT\_BASKET.当变量被删除这种事件发生时，要求这样做,否则若$FRUIT\_BASKET未引用，则当其被删除时产生出错信息：

> test: argument expected

由于Shell没有引用$FRUIT.BASKET的空值，所以就会存在这样的出错信息,这种情况下,该测试语句将类似于:
[-z ]

由于丢失了字符串参数，test命令提示用户丢失了一个必需的参数.通过对$FRUIT_BASKET的引用，该test命令将类似于:

[-z ” “]

这里，必需的字符串参数为” “.

**检查是否两个字符串相等**

test命令可以确认两个字符串是否相等．若两个字符串包含确切相同的字符，则主为是相等的，如，字符串：

> “There are more things in heaven and earth”
>
> “There are more things in heaven and earth”

是相等的，但字符串

> “than are dreamt of in your philosophy”
>
> “Than are dreamt of in your Philosophy”

由于在首字母大小写不同，所以是不相等的．

检查两个字符串是否相等的基本语法是：

> test string1 = string2
>
> 或
>
> [ string1=string2 ]

这里，string1和string2是两个要比较的字符串.

下面给出一个使用字符串进行比较的一个简单例子：

> if  [ “$FRUIT” = apple　]; then
> echo “An apple a day keeps the doctor away.”
> else
> echo “You must like doctors,your fruit $FRUIT is not an apple.”
> fi

若用　”!=”操作符代替 “=”,则test在两个字符串不等时则返回真,可用 “!=”重写前面的命令如下：

> if [ “$FRUIT” != apple ] ; then
> echo “You must like doctors,your fruit $FRUIT is not an apple.”
> else
> echo “An apple a day keeps the doctor away.”
> fi

**3.数字比较**



还有，比较字符串相等用[ “$a” = “$b” ],而[ $a -eq $b ]是比较数值型变量a b用的.

待续… 　见 UNIX.Shell编程24学时教程.pdf　第101页