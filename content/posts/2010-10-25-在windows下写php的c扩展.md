---
title: 在Windows下写PHP的C扩展
author: admin
type: post
date: 2010-10-25T05:22:06+00:00
url: /archives/6328
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php扩展

---
安装好如下软件：

1. VC++ 6

2. php二进制环境

3. Cygwin.


**I. 下载php的源码包**

下载后php源码包解压后有个ext目录，这个目录就是负责开发扩展的目录，目录中有默认你扩展的所有源码。还有两个重要的文件：ext\_skel , ext\_skel_win32.php.

ext_skel是创建扩展的shell，在windows上无法运行，所以就必须要有Cygwin。

**II. 建立php扩展骨架目录文件**

如果你的cygwin没有安装在c:\cygwin，进入php源码包\ext目录下,修改ext\_skel\_win32.php :

$cygwin_path = ‘c:\cygwin\bin’;

修改为你的cygwin目录

$cygwin_path = ‘d:\cygwin\bin’;

命令行方式进入ext目录然后运行：

php ext\_skel\_win32.php –extname=myhello

（当然，为了保证上面的命令行能正常运行，首先你得确保你的php目录在系统的环境变量里）

运行该命令后，有人发现下面的错误

Warning: fopen(myhello/myhello.dsp): failed to open stream: No such file or directory in D:\cygwin\php-5.2.6\ext\ext_skel_win32.php on line 45

Warning: fopen(myhello/myhello.php): failed to open stream: No such file or directory in D:\cygwin\php-5.2.6\ext\ext_skel_win32.php on line 52


说明你的 cygwin 安装不完整。要是没报错你的myhello扩展就创建成功了。这就是一个简单的扩展框架，用纯c语言编写。

**III. 添加依赖的php5ts.lib**

在php的二进制包中的 dev目录下将 php5ts.lib 拷到我们的myhello目录中, 否则编译将通不过。

**IV. 添加hello c代码**

生成的myhello目录中有关键文件包括

myhello.dsp,

myhello.c,

php_myhello.h,

其他文件暂时不必关心.

提示：切忌myhello目录不可以挪移出ext目录,否则会编译报缺少php.h.

 1. 修改php_myhello.h

扩展的新函数: 在PHP\_FUNCTION(confirm\_myhello_compiled); 行后添加一行

> PHP_FUNCTION(confirm_myhello_compiled);
>
> PHP_FUNCTION(myhello);  // 新增的行

2. 修改myhello.c

在PHP\_FUNCTION(confirm\_myhello_compiled) 后添加我们的新函数

> PHP_FUNCTION(myhello){
>
> php_printf(”Hello C extension”);
>
> }

在数组zend\_function\_entry myhello_functions[]增加一行

> zend_function_entry myhello_functions[] = {
>
> PHP_FE(confirm_myhello_compiled,    NULL)        /* For testing, remove later. */
>
> PHP_FE(myhello, NULL) // 新增的行
>
> {NULL, NULL, NULL}    /* Must be the last line in myhello_functions[] */
>
> };

**V. 构建DLL文件**

用vc6打开我们的工程，就是myhello.dsp

1. 修改编译方式为release: 选择Build->Set Active Configuration设置默认编译方式为Release, 否则会提示缺少php5ts_debug.lib ，其实就是php5ts.lib。

2. 按F5编译。会在ext上级的Release\_TS目录下生成php\_myhello.dll

提示：如果愿意使用命令行编译也是可以的，命令如下：

msdev myhello\myhello.dsp /MAKE “myhello – Win32 Release_TS”

**VI. 集成dll到php中。**

1. 把我们生成的 php_myhello.dll放到二进制php环境的ext目录下.

2. 然后修改php.ini, 添加 extension=php_myhello.dll 重启apahce。

3. 新建c_test.php 内容为

在浏览器里打开会看到页面：

>

> hello C extension.
>