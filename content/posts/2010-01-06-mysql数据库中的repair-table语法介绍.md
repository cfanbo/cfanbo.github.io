---
title: MySQL数据库中的REPAIR TABLE语法介绍
author: admin
type: post
date: 2010-01-06T01:56:48+00:00
url: /archives/2795
IM_contentdowned:
 - 1
categories:
 - MySQL

---
REPAIR [LOCAL | NO\_WRITE\_TO_BINLOG] TABLE
[pre] tbl\_name[,tbl\_name] … [QUICK] [EXTENDED] [USE_FRM]

REPAIR TABLE用于修复被破坏的表。默认情况下，REPAIR TABLE与 myisamchk –recovertbl_name具有相同的效果。REPAIR TABLE对MyISAM和ARCHIVE表起作用。    通 常，您基本上不必运行此语句。但是，如果灾难发生，REPAIR TABLE很有可能从MyISAM表中找回所有数据。如果您的表经常被破坏，您应该尽力 找到原因，以避免使用REPAIR TALBE。请参见A.4.2节，“如果MySQL依然崩溃，应作些什么”。同时也见15.1.4节，“MyISAM 表方面的问题”。
本语句会返回一个含有以下列的表：
![20094147086](http://blog.haohtml.com/wp-content/uploads/2010/01/20094147086.jpg)

对 于每个被修复的表，REPAIR TABLE语句会产生多行的信息。上一行含有一个Msg\_type状态值。Msg\_test通常应为OK。如果您没有得 到OK，您应该尝试使用myisamchk –safe-recover修复表，因为REPAIR TABLE尚不会执行所有的myisamchk选 项。我们计划在将来使它的灵活性更强。
如果给定了QUICK，则REPAIR TABLE会尝试只修复索引树。这种类型的修复与使用myisamchk –recover –quick相似。
如果您使用EXTENDED，则MySQL会一行一行地创建索引行，代替使用分类一次创建一个索引。这种类型的修复与使用myisamchk –safe-recover相似。
对 于REPAIR TABLE，还有一种USE\_FRM模式可以利用。如果。MYI索引文件缺失或标题被破坏，则使用此模式。在这种模式下，MySQL可以 使用来自。frm文件重新创建。MYI文件。这种修复不能使用myisamchk来完成。 注释：只能在您不能使用常规REPAIR模式是，才能使用此模 式。。MYI标题包含重要的表元数据（特别是，当前的AUTO\_I [NCRE](http://www.educity.cn/incsearch/search.asp?key=NCRE) MENT值和Delete链接）。这些元数据在REPAIR…USE\_FRM中丢失。如果表被压缩，则不能使用USE\_FRM。因为本信息也 [存储](http://www.educity.cn/incsearch/search.asp?key=%B4%E6%B4%A2) 在。MYI文件中。
REPAIR TABLE语句被写入二进制日志中，除非使用了自选的NO\_WRITE\_TO_BINLOG关键词（或其别名LOCAL）。
警告：如果在REPAIR TABLE运行过程中， [服务器](http://www.educity.cn/incsearch/search.asp?key=%B7%FE%CE%F1%C6%F7) 停 机，则在重新启动之后，在执行其它操作之前，您必须立刻对表再执行一个REPAIR TABLE语句。（通过制作一个备份来启动是一个好办法。）再最不利 情况下，您可以有一个新的干净的索引文件，不含有关数据文件的信息。然后，您执行的下一个操作会覆盖数据文件。这很少发生，但是是有可能的。