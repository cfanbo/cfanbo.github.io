---
title: linux中的shell重定向
author: admin
type: post
date: 2011-07-21T02:40:18+00:00
url: /archives/10548
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - 重定向
 - shell

---
下面的shell 可不可以详细解释一下呀？
(ls you no 2>&1;ls yes 2>&1) 2>&1|egrep * >file
(ls you no 2>&1;ls yes 2>&1)|egrep * >file
(ls you no;ls yes) 2>&1|egrep * >file

2>&1又是什么意思呀？？
在 shell中 >代表输出重定向

```
0表示标准输入
1表示标准输出(默认值)
2表示标准错误输出
2>&1意思是:把 标准错误输出 重定向到 标准输出.
```

ls xxx >out.txt 2>&1, 实际上可换成 ls xxx **1**>out.txt 2>&1；重定向符号>默认是1,错误和输出都传到out.txt了。

|:是管道,例子:
cmd1 | cmd2 意思是:命令cmd1的标准输出座位cmd2的标准输入.

详细解释第三个命令行,(ls you no;ls yes) 2>&1|egrep * >file:
2>&1意思是:把 标准错误输出 重定向到 标准输出.
|意思是:管道;
egrep * :搜索所有的字符串.
>file: 把标准输出导入文件file(如果file存在则清空file,不存在则创建.);