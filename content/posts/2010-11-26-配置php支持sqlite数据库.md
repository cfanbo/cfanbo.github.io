---
title: 配置php支持sqlite数据库
author: admin
type: post
date: 2010-11-26T16:55:32+00:00
url: /archives/6783
IM_contentdowned:
 - 1
categories:
 - 服务器

---
php5-windows默认已经包含了sqlite模块，但没有启用

如果需要支持sqlite，则修改php.ini

启用三个扩展语句

extension=php_pdo.dll

extension=php\_pdo\_sqlite.dll

extension=php_sqlite.dll （可能不是必须的，但最好一起啦）

我第一次只开了下面两个，因此始终无法启用，测试程序会报错

Fatal error: Call to undefined function sqlite_open()

也就是sqlite没有成功启动