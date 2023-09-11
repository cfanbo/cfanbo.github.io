---
title: Innodb中Page结构
author: admin
type: post
date: 2017-01-20T03:50:29+00:00
url: /archives/17377
categories:
 - MySQL

---
 1. 一个存放记录(row)的page，由page header、page trailer、page body组成。如下图:[2]
 [![](http://blog.haohtml.com/wp-content/uploads/2017/01/mysql_page_struct.png)][1]
 [![](http://blog.haohtml.com/wp-content/uploads/2017/01/page_struct.png)][2]

page的完整结构

[![](http://blog.haohtml.com/wp-content/uploads/2017/01/page_full_struct.jpg)][3]

page的结构详情参看如下：

from: [http://forge.mysql.com/wiki/MySQL_Internals_InnoDB#InnoDB_Page_Structure](http://forge.mysql.com/wiki/MySQL_Internals_InnoDB#InnoDB_Page_Structure) High-Altitude Picture

The chart below shows the three parts of a physical record.

**Name****Size**
 Field Start Offsets

 (F*1) or (F*2) bytes

 Extra Bytes

 6 bytes

 Field Contents

 depends on content


Legend: The letter ‘F’ stands for ‘Number Of Fields’.

The meaning of the parts is as follows:

 * The FIELD START OFFSETS is a list of numbers containing the information “where a field starts”.
 * The EXTRA BYTES is a fixed-size header.
 * The FIELD CONTENTS contains the actual data.

**更多查看： [http://www.cnblogs.com/Arlen/articles/1768586.html](http://www.cnblogs.com/Arlen/articles/1768586.html)**

更多参考： [http://www.2cto.com/database/201412/365376.html](http://www.2cto.com/database/201412/365376.html)

[smarty和html_QuikForm联合编程][4]

 [1]: http://blog.haohtml.com/wp-content/uploads/2017/01/mysql_page_struct.png
 [2]: http://blog.haohtml.com/wp-content/uploads/2017/01/page_struct.png
 [3]: http://blog.haohtml.com/wp-content/uploads/2017/01/page_full_struct.jpg
 [4]: http://hedengcheng.com/?p=118