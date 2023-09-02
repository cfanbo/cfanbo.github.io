---
title: mysql中使用 MYSQLBINLOG 来恢复数据
author: admin
type: post
date: 2010-01-24T09:55:33+00:00
excerpt: |
 今天在家里做了一下试验，终于搞明白了以前做复制的时候没有搞明白的问题。原来BINLOG就是一个记录SQL语句的过程，和普通的LOG一样。不过只是她是二进制存储，普通的是十进制存储罢了。
 1、配置文件里要写的东西：
 [mysqld]
 log-bin=yueliangdao_binglog(名字可以改成自己的，如果不改名字的话，默认是以主机名字命名)
 重新启动MSYQL服务。
 二进制文件里面的东西显示的就是执行所有语句的详细记录，当然一些语句不被记录在内，要了解详细的，见手册页。

 2、查看自己的BINLOG的名字是什么。
 show binlog events;
url: /archives/2890
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---
今天在家里做了一下试验，终于搞明白了以前做复制的时候没有搞明白的问题。原来BINLOG就是一个记录SQL语句的过程，和普通的LOG一样。不过只是她是二进制存储，普通的是十进制存储罢了。
1、配置文件里要写的东西：
[mysqld]
log-bin=yueliangdao_binglog(名字可以改成自己的，如果不改名字的话，默认是以主机名字命名)重新启动MSYQL服务。

二进制文件里面的东西显示的就是执行所有语句的详细记录，当然一些语句不被记录在内，要了解详细的，见手册页。2、查看自己的BINLOG的名字是什么。
show binlog events;

### query result(1 records)

 Log_name

 Pos

 Event_type

 Server_id

 End_log_pos

 Info

 yueliangdao_binglog.000001

 4

 Format_desc

 1

 106

 Server ver: 5.1.22-rc-community-log, Binlog ver: 4


3、我做了几次操作后，她就记录了下来。

又一次 show binlog events 的结果。

### query result(4 records)

 Log_name

 Pos

 Event_type

 Server_id

 End_log_pos

 Info

 yueliangdao_binglog.000001

 4

 Format_desc

 1

 106

 Server ver: 5.1.22-rc-community-log, Binlog ver: 4

 yueliangdao_binglog.000001

 106

 Intvar

 1

 134

 INSERT_ID=1

 yueliangdao_binglog.000001

 134

 Query

 1

 254

 use `test`; create table a1(id int not null auto_increment primary key, str varchar(1000)) engine=myisam

 yueliangdao_binglog.000001

 254

 Query

 1

 330

 use `test`; insert into a1(str) values (‘I love you’),(‘You love me’)

 yueliangdao_binglog.000001

 330

 Query

 1

 485

 use `test`; drop table a1
 4、用mysqlbinlog 工具来显示记录的二进制结果，然后导入到文本文件，为了以后的恢复。

详细过程如下：
D:\LAMP\MYSQL5\data>mysqlbinlog –start-position=4 –stop-position=106 yueliangd
ao_binglog.000001 > c:\\test1.txt
test1.txt的文件内容：/*!40019 SET @@session.max_insert_delayed_threads=0*/;

/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;

DELIMITER /*!*/;

# at 4

#7122 16:9:18 server id 1  end_log_pos 106     Start: binlog v 4, server v 5.1.22-rc-community-log created 7122 16:9:18 at startup

# Warning: this binlog was not closed properly. Most probably mysqld crashed writing it.

ROLLBACK/*!*/;

DELIMITER ;

# End of log file

ROLLBACK /* added by mysqlbinlog */;

/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;第二行的记录：D:\LAMP\MYSQL5\data>mysqlbinlog –start-position=106 –stop-position=134 yuelian

gdao_binglog.000001 > c:\\test1.txt

test1.txt内容如下：

/*!40019 SET @@session.max_insert_delayed_threads=0*/;

/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;

DELIMITER /*!*/;

# at 106

#7122 16:22:36 server id 1  end_log_pos 134     Intvar

SET INSERT_ID=1/*!*/;

DELIMITER ;

# End of log file

ROLLBACK /* added by mysqlbinlog */;

/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;

第三行记录：
D:\LAMP\MYSQL5\data>mysqlbinlog –start-position=134 –stop-position=254 yuelian

gdao_binglog.000001 > c:\\test1.txt

内容：

/*!40019 SET @@session.max_insert_delayed_threads=0*/;

/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;

DELIMITER /*!*/;

# at 134

#7122 16:55:31 server id 1  end_log_pos 254     Query    thread_id=1    exec_time=0    error_code=0

use test/*!*/;

SET TIMESTAMP=1196585731/*!*/;

SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=1, @@session.unique_checks=1/*!*/;

SET @@session.sql_mode=1344274432/*!*/;

/*!\C utf8 *//*!*/;

SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=33/*!*/;

create table a1(id int not null auto_increment primary key,

str varchar(1000)) engine=myisam/*!*/;

DELIMITER ;

# End of log file

ROLLBACK /* added by mysqlbinlog */;

/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;/*!40019 SET @@session.max_insert_delayed_threads=0*/;

第四行的记录：

D:\LAMP\MYSQL5\data>mysqlbinlog –start-position=254 –stop-position=330 yuelian

gdao_binglog.000001 > c:\\test1.txt/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;

DELIMITER /*!*/;

# at 254

#7122 16:22:36 server id 1  end_log_pos 330     Query    thread_id=1    exec_time=0    error_code=0

use test/*!*/;

SET TIMESTAMP=1196583756/*!*/;

SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=1, @@session.unique_checks=1/*!*/;

SET @@session.sql_mode=1344274432/*!*/;

/*!\C utf8 *//*!*/;

SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=33/*!*/;

use `test`; insert into a1(str) values (‘I love you’),(‘You love me’)/*!*/;

DELIMITER ;

# End of log file

ROLLBACK /* added by mysqlbinlog */;

/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;5、查看这些东西是为了恢复数据，而不是为了好玩。所以我们最中还是为了要导入结果到MYSQL中。D:\LAMP\MYSQL5\data>mysqlbinlog –start-position=134 –stop-position=330 yuelian

gdao_binglog.000001 | mysql -uroot -p

或者

D:\LAMP\MYSQL5\data>mysqlbinlog –start-position=134 –stop-position=330 yuelian

gdao_binglog.000001 >test1.txt

进入MYSQL导入

mysql> source c:\\test1.txt

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Database changed

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Charset changed

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.03 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

6、查看数据：

mysql> show tables;

+—————-+

| Tables_in_test |

+—————-+

| a1             |

+—————-+

1 row in set (0.01 sec)

mysql> select * from a1;

+—-+————-+

| id | str         |

+—-+————-+

|  1 | I love you  |

|  2 | You love me |

+—-+————-+

2 rows in set (0.00 sec)来源： [http://blog.chinaunix.net/u/29134/showart_434296.html](http://blog.chinaunix.net/u/29134/showart_434296.html)