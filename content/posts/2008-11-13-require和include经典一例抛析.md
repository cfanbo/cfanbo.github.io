---
title: require和include经典一例抛析
author: admin
type: post
date: 2008-11-13T01:45:49+00:00
excerpt: |
 在php中，include和require的作用比较容易混淆。下面我以一个经典例子来深刻说明它们的区别。
 当我们经常访问一个数据库时，可以把连库语句写成一个文件
 我们可以通过把require或include放在函数里面来解决这个问题。
 如果用include，文件的第一个函数调用处将顺利通过，但第二个调用将无法执行，原因是不能在没有关闭数据库时在打开一次，也就是说，con_db.php3执行了两次。将include换成require,一切都正常。
 也就是说，require类似于一次预扫描，在程序执行时，无论在函数里或是函数外，都将先把require的文件执行，且只执行一次。而include则是每执行一次就调用一次文件，即这次执行后，下次再执行执行到这里，仍将再执行一次。
url: /archives/607
IM_contentdowned:
 - 1
categories:
 - 程序开发

---
    在php中，include和require的作用比较容易混淆。下面我以一个经典例子来深刻说明它们的区别。
    当我们经常访问一个数据库时，可以把连库语句写成一个文件
con_db.php3

在实际应用时，我们可以在程序中调用这个文件。
如require(“con\_db.php3”)或include(“con\_db.php3)
这时，两个函数的效果是差不多的。
但如果这样用
filename.php3

文件到myfun处将不能继续执行，因为函数里无法得到外面的变量(include也是一样的)。除非把$dbh作为一个变量传给函数。这又增加了调用函数的复杂度。
我们可以通过把require或include放在函数里面来解决这个问题。
如果用include，文件的第一个函数调用处将顺利通过，但第二个调用将无法执行，原因是不能在没有关闭数据库时在打开一次，也就是说，con_db.php3执行了两次。将include换成require,一切都正常。
    也就是说，require类似于一次预扫描，在程序执行时，无论在函数里或是函数外，都将先把require的文件执行，且只执行一次。而include则是每执行一次就调用一次文件，即这次执行后，下次再执行执行到这里，仍将再执行一次。
    因此，如果在一个循环中，某些语句你只想执行一次，那你用require包括它们就可以了。