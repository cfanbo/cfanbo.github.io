---
title: sqlserver日志删除
author: admin
type: post
date: 2011-02-10T02:51:15+00:00
url: /archives/7663
IM_contentdowned:
 - 1
categories:
 - 数据库
tags:
 - mssql

---
长期积累，sqlserver数据库日志文件会变的非常大，确保日志文件不再使用的前提下，可以删除日志文件，对数据库瘦身。具体方法如下：

1. 打开SQL查询分析器，选择数据库，键入：dump transaction db\_name with no\_log，运行即可。

2. setp1完成后，使用企业管理器，打开数据库，右击当前数据库→所有任务→收缩数据库→点击文件→下拉选择数据文件、日志文件→点选“收缩文件至”选项，指定文件大小，确定即可。

注意：此操作，收缩数据库，文件大小的调整，sqlserver会指定一个最小文件大小，设定值不能超过此值。

其它方法请参考: