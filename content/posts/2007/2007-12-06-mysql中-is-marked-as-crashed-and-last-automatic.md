---
title: mysql中 is marked as crashed and last (automatic?)
author: admin
type: post
date: 2007-12-06T20:33:22+00:00
url: /archives/229
IM_data:
 - 'a:1:{s:51:"/fckeditor/editor/images/smiley/msn/teeth_smile.gif";s:51:"/fckeditor/editor/images/smiley/msn/teeth_smile.gif";}'
IM_contentdowned:
 - 1
categories:
 - 数据库

---
使用php+mysql时,用的数据库偶然一次出现了Table ‘./\****/tbl\_admin is marked as crashed and last (automatic?) repair failed],在此以前使用过一次myisamchk ,结果就出现这个错误提示了,在网上也找不了少办法,但都差不多,myisamchk 表名,可是提示tbl\_admin.MYII不存在,在数据库目录里发现这个文件没有了,但同时多出一个tbl_admin.TMD文件,网上查了一下说是一个临时文件的,其实这些是本地的数据,要不要无所谓的,但这个问题我们得解决吧.后来用了以下方面的:

先把这个文件做个备份,然后直接把.TMD扩展名改成.MYI了.然后用check table 命令结果成功了,呵呵,大家如果遇到此类问题不妨一试的.![](/fckeditor/editor/images/smiley/msn/teeth_smile.gif)

另个也可以用其它命令试一下:repair table 表名 和 check table 表名等其它命令的…