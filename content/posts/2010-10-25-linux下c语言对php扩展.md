---
title: '[教程]Linux下C语言对PHP扩展'
author: admin
type: post
date: 2010-10-25T05:33:17+00:00
url: /archives/6332
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php扩展

---
**一，搭建php环境**

下载php 5.2.6 源码 并解压编译安装，搭建php环境

**二，创建扩展项目**

进入源码目录

> cd php5.2.6/ext/
>
> ./ext\_skel –extname=my\_ext

创建名字为my_ext的项目，最终会生成 my_ext.so

**三，更改配置和程序**

> $ vi ext/my_ext/config.m4

根据你自己的选择将

dnl PHP\_ARG\_WITH(my\_ext, for my\_ext support,

dnl Make sure that the comment is aligned:

dnl [ –with-my\_ext Include my\_ext support])

修改成

> PHP\_ARG\_WITH(my\_ext, for my\_ext support,
>
> Make sure that the comment is aligned:
>
> [ –with-my\_ext Include my\_ext support])

或者将

dnl PHP\_ARG\_ENABLE(my\_ext, whether to enable my\_ext support,

dnl Make sure that the comment is aligned:

dnl [ –enable-my\_ext Enable my\_ext support])

修改成

> PHP\_ARG\_ENABLE(my\_ext, whether to enable my\_ext support,
>
> Make sure that the comment is aligned:
>
> [ –enable-my\_ext Enable my\_ext support])

> $ vi ext/my\_ext/php\_my_ext.h

将

PHP\_FUNCTION(confirm\_my\_ext\_compiled); /\* For testing, remove later. \*/

更改为

> PHP\_FUNCTION(say\_hello);

$ vi ext/my_ext/my_ext.c

将

zend\_function\_entry php5cpp_functions[] = {

PHP\_FE(confirm\_my\_ext\_compiled, NULL) /\* For testing, remove later. \*/

{NULL, NULL, NULL} /\* Must be the last line in php5cpp_functions[] \*/

};

更改为

> zend\_function\_entry php5cpp_functions[] = {
>
> PHP\_FE(say\_hello, NULL)
>
> {NULL, NULL, NULL} /\* Must be the last line in php5cpp_functions[] \*/
>
> };

在最后添加：

> PHP\_FUNCTION(say\_hello)
> {
> zend_printf(“hello world\n”);
> }

**四，编译**

> $ cd my_ext
>
> $ /usr/local/php/bin/phpize
>
> ps: 如果出现：Cannot find autoconf.……的错误信息，则需要安装 autoconf (安装过程略)
>
> $ ./configure –with-php-config=/usr/local/php/bin/php-config
>
> $ make

这时会编译出 my\_ext/modules/my\_ext.so

**五，配置php.ini**

将my_ext.so放入/usr/local/php/ext/目录

> $ cp ./modules/my_ext.so /usr/local/php/ext/

$ vi php.ini

修改添加如下：

> extension_dir = ‘/usr/local/php/ext/’
>
> extension=my_ext.so

**六，测试**

$ vi test.php

> $ /usr/local/php/bin/php test.php
> hello world.

则大功告成