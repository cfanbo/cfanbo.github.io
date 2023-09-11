---
title: 很有用的mysqladmin命令
author: admin
type: post
date: 2011-07-14T04:37:02+00:00
url: /archives/10416
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql
 - mysqladmin

---
> [root@localhost ~]$ uname -r
> 2.6.9-22.ELsmp
>
> [root@localhost ~]$ /usr/local/mysql/bin/mysqladmin version -uroot -p
> Enter password:
> /usr/local/mysql/bin/mysqladmin  Ver 8.42 Distrib 5.1.22-rc, for redhat-linux-gnu on x86_64
> Copyright (C) 2000-2006 MySQL AB
> This software comes with ABSOLUTELY NO WARRANTY. This is free software,
> and you are welcome to modify and redistribute it under the GPL license
>
> Server version          5.1.22-rc-log
> Protocol version        10
> Connection              Localhost via UNIX socket
> UNIX socket             /tmp/mysql.sock
> Uptime:                 74 days 7 hours 2 min 31 sec
>
> Threads: 55  Questions: 5612872391  Slow queries: 55749  Opens: 41379  Flush tables: 1  Open tables: 1024  Queries per second avg: 874.422

#top -p pid
按1

术里，几个核里有一个CPU是负载CPU任务调度的，所以会看到经常有一个CPU的负载比别的CPU高。
摘自　：==========================================

**mysqladmin 适合于linux和windows系统**

> ****inux下：mysqladmin -u[username] -p[password] status

windows下：先在安装目录找到mysqladmin.exe，然后在dos界面下change到这个目录，执行

> mysqladmin -u[username] -p[password] extended-status

这里的extended-status 和status只是mysqladmin的两个参数而已！

**MySQLAdmin用法**
用于执行管理性操作。语法是：
**shell> mysqladmin [OPTIONS] command [command-option] command …**
通过执行mysqladmin –help，你可以得到你mysqladmin的版本所支持的一个选项列表。
目前mysqladmin支持下列命令：

> create databasename 创建一个新数据库
> drop databasename 删除一个数据库及其所有表
> extended-status 给出服务器的一个扩展状态消息
> flush-hosts 洗掉所有缓存的主机
> flush-logs 洗掉所有日志
> flush-tables 洗掉所有表
> flush-privileges 再次装载授权表(同reload)
> kill id,id,… 杀死mysql线程
> password 新口令，将老口令改为新口令
> ping 检查mysqld是否活着
> processlist 显示服务其中活跃线程列表
> reload 重载授权表
> refresh 洗掉所有表并关闭和打开日志文件
> shutdown 关掉服务器
> status 给出服务器的简短状态消息
> variables 打印出可用变量
> version 得到服务器的版本信息

所有命令可以被缩短为其唯一的前缀。例如：
shell> mysqladmin proc stat

```
+----+-------+-----------+----+-------------+------+-------+------+
| Id | User  | Host      | db | Command     | Time | State | Info |
+----+-------+-----------+----+-------------+------+-------+------+
| 6  | monty | localhost |    | Processlist | 0    |       |      |
+----+-------+-----------+----+-------------+------+-------+------+
```

Uptime: 10077 Threads: 1 Questions: 9 Slow queries: 0 Opens: 6 Flush tables: 1
Open tables: 2 Memory in use: 1092K Max memory used: 1116K

**mysqladmin status命令结果有下述列：**
Uptime MySQL服务器已经运行的秒数
Threads 活跃线程（客户）的数量
Questions 从mysqld启动起来自客户问题的数量
Slow queries 已经超过long\_query\_time秒的查询数量
Opens mysqld已经打开了多少表
Flush tables flush …, refresh和reload命令数量
Open tables 现在被打开的表数量
Memory in use 由mysqld代码直接分配的内存(只有在MySQL用–with-debug编译时可用)
Max memory used 由mysqld代码直接分配的最大内存(只有在MySQL用–with-debug编译时可用)