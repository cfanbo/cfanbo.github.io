---
title: 图解”How MySQL Replication Works”
author: admin
type: post
date: 2010-06-27T10:27:17+00:00
url: /archives/4148
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - 集群
 - mysql

---

[![](http://blog.haohtml.com/wp-content/uploads/2010/06/3455232156_dc09e11b22_o.jpg)](http://blog.haohtml.com/wp-content/uploads/2010/06/3455232156_dc09e11b22_o.jpg)

[replication by orczhou, on Flickr](http://www.flickr.com/photos/26825745@N06/3455232156/ "replication by orczhou, on Flickr")

在使用MySQL的应用中，如果你的MySQL Server压力逐渐增大，在应用层优化已经到了一定瓶颈时，那么你应该首先考虑 [MySQL Replication](http://dev.mysql.com/doc/refman/5.0/en/replication.html)。本文将利用图示的方式简单的描述出MySQL Replication是如何工作的。

**如何同步**

1. 主库将所有的更新操作，写入二进制日志。

2. 从库运行”IO线程”（Slave IO Thread）读取主库的二进制日志。

3. 从库运行”SQL线程”（Slave SQL Thread）执行IO线程（Slave IO Thread）读取的日志中的SQL,从而保持和主库的一致。


**如何分配请求**

1. 目前，这部分需要在应用层实现。

2. 执行更新SQL(UPDATE，INSERT，DELETE)时，请求主库。

3. 执行查询SQL(SELECT)时，请求从库。


所以，当你的应用(Application)SELECT请求所占的比率越大，那么Relication就会越有效。