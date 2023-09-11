---
title: 'mysql主从复制中出现＂Relay log read failure…”错误信息的解决办法[教程]'
author: admin
type: post
date: 2011-10-13T01:51:57+00:00
url: /archives/11651
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql主从

---
今天我的服务器突然停止复制了。因为对这块不是很熟悉，就上网学习了一下，发现了一篇好文章。不敢独享，

和大家来分享一下。

众所周知MySQL5.1的Replication是比较烂的。MySQL的每一个版本更新关于同步方面每次都是可以看到一大堆。但MySQL 5.1性能是比较突出的。所以经不住诱惑使用MySQL 5.1。所以也要经常遇到一些Bug。如：

```
mysql> show slave status\G

*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.10.118
                  Master_User: repl_wu
                  Master_Port: 3306
                Connect_Retry: 30
              Master_Log_File: mysql-bin.005121
          Read_Master_Log_Pos: 64337286
               Relay_Log_File: relay-bin.003995
                Relay_Log_Pos: 18446697137031827760
        Relay_Master_Log_File: mysql-bin.005121
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 1594
                   Last_Error: Relay log read failure: Could not parse relay log event entry. The possible reasons are: the master's binary log is corrupted (you can check this by running 'mysqlbinlog' on the binary log), the slave's relay log is corrupted (you can check this by running 'mysqlbinlog' on the relay log), a network problem, or a bug in the master's or slave's MySQL code. If you want to check the master's binary log or slave's relay log, you will be able to know their names by issuing 'SHOW SLAVE STATUS' on this slave.
                 Skip_Counter:
          Exec_Master_Log_Pos: 4
              Relay_Log_Space: 64337901
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos:
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno:
                Last_IO_Error:
               Last_SQL_Errno: 1594

               Last_SQL_Error: Relay log read failure: Could not parse relay log event entry. The possible reasons are: the master's binary log is corrupted (you can check this by running 'mysqlbinlog' on the binary log), the slave's relay log is corrupted (you can check this by running 'mysqlbinlog' on the relay log), a network problem, or a bug in the master's or slave's MySQL code. If you want to check the master's binary log or slave's relay log, you will be able to know their names by issuing 'SHOW SLAVE STATUS' on this slave.

1 row in set (0.00 sec)
```

从上面可以看到是中继日值或是Master上的日值出问题了。

首先如果是中继日值坏掉，那只需要找到同步的时间点，然后重新同步，这样就可以有新的中继日值了。如果Master上的日值坏了就麻烦了。

从经验来看，这是中继日值出问题了。处理方法：

**需要找到同步的点。**

日志为：
Master\_Log\_File: mysql-bin.005121，
Relay\_Master\_Log_File: mysql-bin.005121
主要以**Relay_Master_Log_File**为准，**Master_Log_File**为参考。

日志执行时间点：

**Exec_Master_Log_Pos**: **4**

那么现在就可以：

```
    mysql>stop slave;
    mysql>change master to Master_Log_File='mysql-bin.005121', Master_Log_Pos=4;
    mysql>start slave;
    mysql>show slave status\G;
```

进行确认。



**建议：**

在使用MySQL-5.1.36以下的版本的同学，请尽快升级到MySQL-5.1.40 & MySQL-5.1.37sp1

[点击查看原文…][1]

 [1]: http://www.cnblogs.com/niniwzw/archive/2010/02/04/1663685.html