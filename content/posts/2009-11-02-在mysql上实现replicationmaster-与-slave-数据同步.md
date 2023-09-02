---
title: (精典教程)在MySql上实现Replication(Master 与 Slave 数据同步)
author: admin
type: post
date: 2009-11-02T06:38:18+00:00
excerpt: |
 1： 首先确定Master和Slave的数据库版本，Master数据库的版本不能高于Slave数据的版本。

 这里我是使用MySql 5.0.27 作为Master数据库，MySql 6.0.3（alpha）作为Slave进行测试。
url: /archives/2546
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - 集群
 - mysql
 - 主从复制

---
Master: 192.168.1.200
slave: 192.168.1.201

1： 首先确定Master和Slave的数据库版本，Master数据库的版本不能高于Slave数据的版本。

这里我是使用MySql 5.0.27 作为Master数据库，MySql 6.0.3（alpha）作为Slave进行测试。

2：首先在Master数据库的配置文件my.ini (windows)里添加log-bin 和 server-id 两项

eg: [mysqld]

**server-id = 1  //服务器编号**

**log-bin = mysql-bin  //使用的二进制日志文件名
binlog-do-db=test = maindb //同步的数据库(不记录二进制日志)**
**binlog-ignore-db=bbs** **//不允许同步的数据库(不记录二进制日志)**
 **binlog-ignore-db=ceshi   //不允许同步的数据库库(不记录二进制日志)**

在Slave数据库的配置文件my.ini里添加server-id 项

eg:[mysqld]

**server-id = 2
** **replicate-do-db=backup   //告诉slave只做backup数据库的更新(可选)
replicate-ignore-db=bbs //告诉slave忽略这两个数据库的更新(可选**)**
replicate-ignore-db****=ceshi// 同上**

**replicate-ignore-table=_db\_name.tbl\_name //忽略指定数据库里的指定表，多个表重复写此语句_**

·         –replicate-ignore-table=_db\_name.tbl\_name_

告诉从服务器线程不要复制更新指定表的任何语句(即使该语句可能更新其它的表)。要想忽略多个表，应多次使用该选项，每个表使用一次。同–replicate-ignore-db对比，该选项可以跨数据库进行更新。请读取该选项后面的注意事项。

·         –replicate-wild-do-table=_db\_name.tbl\_name_

告诉从服务器线程限制复制更新的表匹配指定的数据库和表名模式的语句。模式可以包含‘%’和‘_’通配符，与LIKE模式匹配操作符具有相同的含义。要指定多个表，应多次使用该选项，每个表使用一次。该选项可以跨数据库进行更新。请读取该选项后面的注意事项 。

如果上面的最后两行不写的话,默认为同步所有数据库

(这里需要理解的是Slave本身也是一个独立的服务器，它作为‘从数据库’是从它通过‘主服务器’日志更新数据角度上理解的。可以把 `server-id` 想象成为IP地址：这些ID标识了整个同步组合中的每个服务器。如果没有指定 `server-id` 的值，如果也没定义 `master-host`，那么它的值就为1，否则为2。注意，如果没有设定 `server-id`，那么master就会拒绝所有的slave连接，同时slave也会拒绝连接到master上。)

3：修改配置后启动Master数据服务。在Master数据库上建立一个用户，用于Slave数据连接以便同步数据。一般来说Slave数据只用于同步数据，所以我们在建立这个用户时只授予它REPLICATION SLAVE 权限。

**GRANT REPLICATION SLAVE ON *.* TO ‘slaver’@’192.168.1.201’ IDENTIFIED BY ‘slaver123’;**

4：在Master数据库上执行

**FLUSH TABLES WITH READ LOCK;**

命令以刷新数据并阻止对Master数据的写入操作。然后将Master数据的data目录复制一份覆盖Slave数据库的data目录，这样Master和Slaver就有了相同的数据库了。在复制时可能不需要同步 mysql 数据库，因为在slave上的权限表和master不一样。这时，复制的时候要排除它。同时不能包含任何\`master.info~ 或 \`relay-log.info\` 文件。覆盖好后执行:

mysql> **UNLOCK TABLES;**

释放锁定。

5：在Master数据库上执行

**SHOW MASTER STATUS;**

查看当前Master数据库上的一些我们将要使用的信息：

[![mysql](http://blog.haohtml.com/wp-content/uploads/2009/11/clip_image002.jpg)][1]

File 表示 Master用于记录更新数据操作的日志文件，Position 表示当前日志的记录位置，这也是Slave 需要开始同步数据的位置。

6：启动Slave数据库 执行：（这点连接Master数据库所要的参数）

**mysql> CHANGE MASTER TO
** ->  **MASTER_HOST=’192.168.1.200′,** //Master服务器地址
->  **MASTER_USER=’slaver’,** //Slave服务器更新时连接Master使用的用户名
->  **MASTER_PASSWORD=’slaver’,** // Slave服务器更新时连接Master使用的密码
->  **MASTER\_LOG\_FILE=’mysql-bin.000004′,** //更新操作日志
->  **MASTER\_LOG\_POS=837016;** //同步数据的开始位置

**change master to  master_host=’192.168.1.200′,master_user=’slaver’, master_password=’slaver123′, master_log_file=’mysql-bin.000004′, master_log_pos= 837016;**

_**另一种办法也可以直接在my.ini配置文件里做配置,本人未做测试（此方法只适合第一次配置主从时使用）**_

> 修改B的my.ini文件，在mysqld配置项中加入下面配置：
> **server-id=2
> master-host=10.10.10.22
> master-user=backup #同步用户帐号
> master-password=1234
> master-port=3306
> master-connect-retry=60 预设重试间隔60秒
> 上面的master\_log\_file=’mysql-bin.000004’和master\_log\_pos=837016 这两项不知用不用写的,
> replicate-do-db=backup 告诉slave只做backup数据库的更新
> binlog-ignore-db=bbs,ceshi 告诉slave忽略这两个数据库的更新**
>
> **
>**

上面命令执行完毕后，执行START SLAVE; 命令启动数据更新。在Slave 数据库上执行:

**START SLAVE;**

**SHOW SLAVE STATUS\G;**

查看从数据跟主数据库的连接状态是否正常，如果显示的信息中的** Slave-IO-Running 和 Slave_SQL_Running 值为 yes**，表示用于数据同步的 io线程和sql操作线程已经成功启动。如果发现其中一个值为NO,则说明配置失败，详细原因， [请参考这里](/index.php/archives/2552)

7：到此已经建立Master和Slave数据库的同步了。你可以在Master数据库上更新一个表的数据，然后查看Slave数据库上对应表是否做了相应的更改。

注： slave开始同步后，就能在数据文件目录下找到2个文件 `` `master.info` `` 和 `` `relay-log.info` ``。slave利用这2个文件来跟踪处理了多少master的二进制日志。master.info 记录了slave 连接master进行数据同步的参数，relay-log.info 记录了slave进行数据更新使用的中续日志的的信息。

如果在主从复制中的从服务器中遇到一些无关重要的语句,可以执行以下命令跳过去.

```
mysql> STOP SLAVE;
mysql> SET GLOBAL SQL_SLAVE_SKIP_COUNTER = n;
mysql> START SLAVE;
mysql> show slave status\G;
```

命令见：

也可以在slave服务器的my.cnf文件里加入

```
slave-skip-errors = 1062
```

参考：

**相关教程：**

show slave status 参数详解：**** [http://blog.haohtml.com/archives/6871](http://blog.haohtml.com/archives/6871)

对于Mysql的主从复制更多内容请参考：

使用主从复制方案，最大的问题就是主从之间的复制延时问题，这个可以在使用mysql proxy (lua)脚本插件的时候，利用单独一个表来解决，方案请参考： [http://www.haohtml.com/database/mysql/46714.html](http://www.haohtml.com/database/mysql/46714.html)

 [1]: /wp-content/uploads/2009/11/clip_image002.jpg