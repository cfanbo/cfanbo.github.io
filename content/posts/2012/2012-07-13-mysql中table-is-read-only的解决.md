---
title: mysql中Table is read only的解决办法
author: admin
type: post
date: 2012-07-13T01:51:34+00:00
url: /archives/13158
IM_contentdowned:
 - 1
categories:
 - 数据库
tags:
 - mysql

---
今天遇到一个这样的提示repair数据表的时候出现“mysql中Table is read only”

在mysql中，Select之类的都正常，但在网页程序中提示：Table ‘********’ is read only

然后我

> chmod -R 0777  /var/lib/mysql/taoniu2007/

给数据库目录的所属用户和组改为mysql，并加上777的权限，还是一样提示。

程序中使用root连接，也是一样的提示。

想用myisamchk来检查一下，也提示read only。

最终在这里找到了解决方法： [http://www.mysqltalk.org/re-the-table-is-read-only-vt154092.htm](http://www.mysqltalk.org/re-the-table-is-read-only-vt154092.htm) l

引用一下

> SQL代码
> Hi,
>
> I just encountered a similar problem on one of my production servers
> this morning. (I’m still investigating the cause.) After doing a
> quick bit of Google-searching, this solved my problem:
>
> mysqladmin -u  -p flush-tables
>
> By the way: All directories in /var/lib/mysql should have 700
> permissions (owned my the mysql user) and everything within those
> directories should be 660 (owned by the mysql user and mysql group).
>
> (This was on a FreeBSD 4.8 server running MySQL Server 3.23.58)
>
> Hope this helps,
> Seth

运行flush-tables后，read only问题解决