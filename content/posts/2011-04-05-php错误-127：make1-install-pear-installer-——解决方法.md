---
title: 'php错误 127：make[1]: *** [install-pear-installer] ——解决方法'
author: admin
type: post
date: 2011-04-05T12:08:50+00:00
url: /archives/9000
IM_contentdowned:
 - 1
categories:
 - 服务器

---
在上一节 [安装Nginx+Mysql+php+memcache](http://blog.haohtml.com/archives/6051) 的时候，在编译、生成php都正常，在安装的时候出现下面错误：

错误如下：

> error while loading shared libraries: libiconv.so.2: cannot open shared object file: No such file or directory
>
> make[1]: \*** [install-pear-installer] 错误 127
>
> make: \*** [install-pear] Error 2

google了下，试了很多方法都不行，find后，发现我的/usr/local/lib里面有库文件，但是/usr/lib里面没有。

于是用软链接：

> **ln -s /usr/local/lib/libiconv.so.2 /usr/lib/libiconv.so.2**

之后 **make clean**

再编译 生成 安装，一切正常。

=================================================

另外出现libmysqlclient.so.15、libmysqlclient.so.16错误，同样可以去/usr/local/mysql/lib下面复制这两个文件到/usr/lib下面，或者直接设置软链接。。。

再make clean，重新编译安装即可。

另外一种办法请参考: