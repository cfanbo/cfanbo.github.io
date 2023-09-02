---
title: 'FATAL ERROR: Could not find ./bin/my_print_defaults的解决办法'
author: admin
type: post
date: 2011-06-06T14:25:48+00:00
url: /archives/9674
IM_contentdowned:
 - 1
categories:
 - 服务器

---
网上很多方法都是：/usr/local/mysql/scripts/mysql\_install\_db –user=mysql

但是很有可能报错，找不到bin目录中的my_print_defaults

错误信息：

>

> FATALERROR:Couldnotfind./bin/my_print_defaults
>

>
>

> If you are using a binary release, you must run this script from
>

>
>

> within the directory the archive extracted into. If you compiled
>

>
>

> MySQL yourself you must run ‘make install’ first.
>

或

>

> [root@bogon scripts]# /usr/local/mysql/scripts/mysql_install_db –user=mysql&
>

>
>

> [1] 16874
>

>
>

> [root@bogon scripts]#
>

>
>

> FATAL ERROR: Could not find ./bin/my_print_defaults
>

>
>

> If you compiled from source, you need to run ‘make install’ to copy the software into the correct location ready for operation.
>

>
>

> If you are using a binary release, you must either be at the top level of the extracted archive, or pass the –basedir option pointing to that location.
>

**解决方法：**

>

> [root@bogon scripts]# /usr/local/mysql/scripts/mysql_install_db –user=mysql –basedir=/usr/local/mysql –datadir=/usr/local/mysql/data &
>

(这点非常重要)

FreeBSD下的解决办法是：

> /usr/local/bin/mysql\_install\_db –user=mysql –basedir=/usr/local/ –datadir=/var/db/mysql &