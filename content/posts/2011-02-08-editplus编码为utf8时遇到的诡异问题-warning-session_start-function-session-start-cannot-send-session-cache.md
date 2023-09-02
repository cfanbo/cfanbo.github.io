---
title: 'editplus编码为utf8时遇到的诡异问题-Warning: session_start() [function.session-start]: Cannot send session cache'
author: admin
type: post
date: 2011-02-08T06:11:16+00:00
url: /archives/7645
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - editplus

---
arning Cannot send session cookie – headers already sent…问题的解决(PHP的UTF-8 BOM引起的问题)

习惯了用edit plus进行php编程,所以有时会出现一些不为人知的错误，很麻烦;。
近日，在开发项目时，某些页面总是出现以下问题：

Warning: session_start() [function.session-start]: Cannot send session cookie – headers already sent by (output started at E:\web\Apache2\htdocs\index.php:1) in E:\web\Apache2\htdocs\functions\sessions.php on line 67

Warning: session_start() [function.session-start]: Cannot send session cache limiter – headers already sent (output started at E:\web\Apache2\htdocs\index.php:1) in E:\web\Apache2\htdocs\functions\sessions.php on line 67
经过详细搜索,得到以下原因:
我的edit plus中设置了默认的编码为utf-8,且UTF_8签名为:总是添加签名;
于是尝试以下操作:
在edit plus 的工具->参数->文件->UTF_8签名一项中,更改选项”总是添加签名”为”总是移除签名”,然后打开index.php文件,并重新另存为,重新运行脚本,终于可以正常了;

另外，在网上找到了两篇比较有参考价值的文章，希望有人碰到此种情况时可以完美解决！

一个UTF-8 BOM引起的PHP的诡异问题
一、
//—a.php

将a.php保存为utf-8格式，结果用浏览器访问这个php文件，就会出现如下错误：
Warning: session_start() [function.session-start]: Cannot send session cache limiter – headers already sent (output started at ×××.php:1) in ×××on line 2

这个问题很常见，多数是因为在session_start之前有输出了！对于老鸟来说，这个错误基本上不会发生，但是如果你是用DW或是editplus等编

辑器写代码的，连高手也有可能发生这个错误！

如上面的提示：在第×××文件的第１行，×××文件的第２行，随你看，这两处是不会有任何输出语句的，很奇怪还是会出错，为什么呢
原来：Unicode 签名 (BOM) 可在文档中包括字节顺序标记 (BOM)。BOM 是位于文本文件开头的 2 到 4 个字节，可将文件标识为 Unicode，如果是这样，还标识后面字节的字节顺序。由于 UTF-8 没有字节顺序，因此可以选择添加 UTF-8 BOM。对于 UTF-16 和 UTF-32，这是必需的。
看见没有！如果选了这个选项，就会在页面的最前面输出2到4个字节！
而 session_start() 要求之前没有任何输出给客户端浏览器

二、另外还有一个地方可能会出错，例如：
/–a.php–
?>
空行
空行

如果你包含a.php之后再来也会有这个问题，通常的建议是经常被包含的文件末尾不要有?>
又如：在调用Session_Start()之前不能有任何输出.例如下面是错误的.
==========================================
1行
2行
==========================================
已经经过试验，事实确实是如此诡异。

三、session\_start()、set\_cookie()、header()前面都加上@应该可以抑制这个警告。

四、在editplus编辑器中，如果先把utf-8的a.php文件转换为gb2312或是其他，然后再转换为utf-8这样就可以成功访问了，也就是说文件开头的BOM被去掉了，这时候的UTF-8 是无BOM类型的了

PHP-关于utf-8编码问题引起的session_start()错误

采用默认的gb2312编码时，兼容Ansi编码，文件头部无任何附加信息，此时session\_start()可以正常工作。采用utf编码时，大部分编辑器都会在在文件头部附加一个BOM块，我的EditPlus附加的是FF FE，用16进制编辑器可以很清楚的看到。这样，当调用session\_start()时，实际上已经向浏览器输出两个字节,只不过是不可见字符浏览器中出现如下警告：
Warning: session_start() [function.session-start]: Cannot send session cookie – headers already sent by (output started at ………………….
解决方法：
1、手动去掉BOM块，可以在16进制编辑器如UltraEdit中编辑，或者采用编辑器自带的功能，好的编辑器一般提供选择是否去除BOM块。
2、自己编写脚本更正，这要针对不同的编辑器，BOM头定义：
UTF-8                           EF BB BF
UTF-16 Big Endian               FE FF
UTF-16 Little Endian            FF FE
UTF-32 Big Endian               00 00     FE FF
UTF-32 Little Endian            FF FE 00 00

摘自：