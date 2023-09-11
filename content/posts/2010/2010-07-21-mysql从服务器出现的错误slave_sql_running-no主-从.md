---
title: 'mysql从服务器出现的错误:Slave_SQL_Running: No(主-从)'
author: admin
type: post
date: 2010-07-21T02:40:27+00:00
url: /archives/4731
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - 集群
 - mysql集群

---
mysql服务器为主-从配置时,发现从MySQL Slave未和主机同步，查看Slave状态：

> mysql> show slave statusG
> Slave\_IO\_Running: Yes
> Slave\_SQL\_Running: No
> Last_Errno: 1062
> ….
> Seconds\_Behind\_Master:NULL

**原因：**
1.程序可能在slave上进行了写操作
2.也可能是slave机器重起后，事务回滚造成的.

**解决办法I：**
1.首先停掉Slave服务：**slave stop**
到主服务器上查看主机状态：
记录File和Position对应的值。
3.到slave服务器上执行手动同步：

> mysql> show master status;
> +——————+———–+————–+——————+
> | File | Position | Binlog\_Do\_DB | Binlog\_Ignore\_DB |
> +——————+———–+————–+——————+
> | mysql-bin.000020 | 135617781 | | |
> +——————+———–+————–+——————+
> 1 row in set (0.00 sec)

> mysql> change master to
> > master\_host=’master\_ip’,
> > master_user=’user’,
> > master_password=’pwd’,
> > master_port=3307,
> > master\_log\_file=’mysql-bin.000020′,
> > master\_log\_pos=135617781;
> 1 row in set (0.00 sec)
> mysql> slave start;
> 1 row in set (0.00 sec)

再次查看slave状态发现：

> Slave\_IO\_Running: Yes
> Slave\_SQL\_Running: Yes
> …
> Seconds\_Behind\_Master: 0

注:这种办法可能会导致从服务器上的数据不完整,如从服务器一直出错,但主服务器日志文件一直在增加,过好长时间,再直接从主服务器上取日志位置,可能会造成错误期间的数据无法更新到从服务器中.这里建议采用下面的这种办法(将错误语句直接跳过).

**解决办法II：**

> mysql> slave stop;
> mysql> set GLOBAL SQL\_SLAVE\_SKIP_COUNTER=1;
> mysql> slave start;

set GLOBAL SQL\_SLAVE\_SKIP_COUNTER=N,用来跳过备机的一条或N条出错的复制语句。然后重新start slave即可。

**1.问题：收到报警，从数据库在同步的过程出现问题，已停止同步。**
分析:查看从数据库的错误日志，找到如下信息:
091229 11:49:41 [ERROR] Error reading packet from server: Got packet bigger than ‘max\_allowed\_packet’ bytes ( server_errno=2020)
解决办法：1.增加/etc/my.cnf 中的max_allowed_packet,增加他的大小，我修改为10M,然后，重启服务器。
2.为了不影响业务，直接在数据库里面，修改max\_allowed\_packet的大小

> mysql>set GLOBAL max\_allowed\_packet=10475520; (注意不能直接写成10M)
> mysql>change master to master\_host=’192.168.1.203′,master\_user=’yang’, master\_password=’yang’, master\_port=3306, master\_log\_file=’mysql-bin.000018′,master\_log\_pos=395332157;

(注：master\_log,master\_log_pos的值要设置成停止同步的那个位置，不然会造成，数据不一致）

> mysql>slave start;

3.在同步过程中，又出现了一个错误:
[ERROR] Slave: Error ‘Duplicate entry ‘18923’ for key 1′ on query.
解决方法：1.查找主数据库对应位置的操作记录

> #/opt/mysql/bin/mysqlbinlog ../log/bin.000009 –start-position=100 –stop-position=110

(分析binlog 日志)

#在从数据库删除重复记录，然后，在change master，OK，问题解决了。

**参考文档:** [mysql集群从服务器复制传递和状态文件(master.info和relay_log.info)](http://blog.haohtml.com/index.php/archives/3463)