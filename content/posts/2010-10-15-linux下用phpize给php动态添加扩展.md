---
title: linux下用phpize给PHP动态添加扩展
author: admin
type: post
date: 2010-10-15T04:28:19+00:00
url: /archives/6118
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux
 - phpize
 - php扩展

---
相关教程： [FreeBSD下安装php扩展](http://blog.haohtml.com/index.php/archives/7001)

使用php的常见问题是编译php时忘记添加某扩展，后来想添加扩展，但是因为安装php后又装了一些东西如PEAR等，不想删除目录重装，这里就需要用到phpize了。

如我想增加bcmath扩展的支持，这是一个支持大整数计算的扩展。windows自带而且内置，linux“本类函数仅在 PHP 编译时配置了 –enable-bcmath 时可用”（引号内是手册中的话）

注意,有些扩展需要和php的版本保持一致才可以的.

解压bcmath包,进入里面的ext/bcmath目录,然后执行/usr/local/php/bin/phpize，phpize在php安装完以后会有这个命令的, 会发现当前目录下多了一些configure文件，然后再执行./configure命令即可.

> #/usr/local/php/bin/phpize
> #./configure –with-php-config=/usr/local/php/bin/php-config

注意要先确保**/usr/local/php/bin/php-config**存在。 (如果你的php安装路径不是默认的,请修改为php安装的路径)

如果没有报错，则make，再make install ，然后它告诉你一个目录.

> #make
> #make install

你把该目录下的bcmath.so拷贝到你php.ini中的extension_dir指向的目录中，

修改php.ini,在最后添加一句

> extension=bcmath.so

重启WEB服务，再执行phpinfo(),惊喜发现：

[![](https://blogstatic.haohtml.com//uploads/2023/09/bcmath.png)][1]

互此bcmath扩展已经安装成功！

这里也有一篇安装php扩展的文章,” [Linux下 安装XCache 扩展”,安装方法点击查看](http://blog.haohtml.com/index.php/archives/6128)



