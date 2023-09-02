---
title: show slave status\G中的Read_Master_Log_Pos和Relay_Log_Pos的(大小)关系
author: admin
type: post
date: 2011-09-26T03:20:56+00:00
url: /archives/11529
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - 主从复制

---
Just to clarify, there are three sets of file/position coordinates in SHOW SLAVE STATUS:

1) The position, ON THE MASTER, from which the I/O thread is reading: Master_Log_File/Read_Master_Log_Pos. —–相对于主库,从库读取主库的二进制日志的位置,是IO线程

2) The position, IN THE RELAY LOGS, at which the SQL thread is executing: Relay_Log_File/Relay_Log_Pos —-相对于从库,是从库的sql线程执行到的位置

3) The position, ON THE MASTER, at which the SQL thread is executing: Relay_Master_Log_File/Exec_Master_Log_Pos —-相对于主库,是从库的sql线程执行到的位置

Numbers 2) and 3) are the same thing, but one is on the slave and the other is on the master.

> mysql > **show slave status \G**
>
> Master\_Log\_File: mysql-bin-m.000329
>
> Read\_Master\_Log_Pos: 863952156　—-上面二行代表IO线程,相对于主库的二进制文件
>
>
>
> Relay\_Log\_File: mysql-relay.003990
>
> Relay\_Log\_Pos: _25077069　—-_上面二行代表了sql线程,相对于从库的中继日志文件
>
> Relay\_Master\_Log_File: mysql-bin-m.000329
>
> …..
>
> Exec\_Master\_Log_Pos: 863936961—上面二行代表了sql线程,相对主库
>
> (为了方便演示,我把上面这行提前了.)
>
> Relay\_Log\_Space: 25092264—当前relay-log文件的大小
>
> Slave\_IO\_Running: Yes
>
> Slave\_SQL\_Running: Yes

从上面可以看到,read\_master\_log\_pos 始终会大于exec\_master\_log\_pos的值(也有可能相等):因为一个值是代表io线程,一个值代表sql线程;sql线程肯定在io线程之后.(当然,io线程和sql线程要读写同一个文件,否则比较就失去意义了) .

在binlog中,Xid代表了提交的事务号.现在我们分别去主从库看看,验证一下,在主库的mysql-bin-m.000329文件的863936961处是否与从库的mysql-relay.003990文件的25077069处有相同的sql语句.

**先看主库的binlog:**

> _# at 863936961_
>
> #100111 20:11:39 server id 115000 end\_log\_pos 863937234 Query thread\_id=515886 exec\_time=0 error_code=0
>
> use mall00/\*!\*/;
>
> UPDATE mall00.t_item_sid88 SET item_end_time = 1263816699, item_is_online = 1, item_status = 1 WHERE iid IN (94322390, 94322428, 94322452, 94322473, 94322506, 94322532, 94322604, 94322641, 94322670, 94322706)/*!*/;
>
> _\# at 863937234_
>
> #100111 20:11:39 server id 115000 end\_log\_pos 863937261 **Xid** = 1225244590
>
> COMMIT/\*!\*/;
>
> \# at 863937261
>
> #100111 20:11:39 server id 115000 end\_log\_pos 863937457 Query thread\_id=515886 exec\_time=0error_code=0
>
> SET TIMESTAMP=1263211899/\*!\*/;

**再看从库的relaylog:**

> _# at 25077069_
>
> #100111 20:11:39 server id 115000 end\_log\_pos 863937234 Query thread\_id=515886 exec\_time=0 error_code=0
>
> use mall00/\*!\*/;
>
> UPDATE mall00.t\_item\_sid88 SET item\_end\_time = 1263816699, item\_is\_online = 1, item_status = 1 WHERE iid IN (94322390, 94322428, 94322452, 94322473, 94322506, 94322532, 94322604, 94322641, 94322670, 94322706)/\*!\*/;
>
> _\# at 25077342_
>
> #100111 20:11:39 server id 115000 end\_log\_pos 863937261 **Xid** = 1225244590
>
> COMMIT/\*!\*/;

从上面的日志中,可以看到binlog与realy-log的内容是相同的,除了开头的at位置处的偏移量.**因为偏移量始终是相对于文件自身来说,主库上相对于binlog本身,从库上相对于relay-log本身.**还可以看到,在每个query语句过后,都有一个**Xid** 的event,提交该事务,也表明在mysql中是自动提交的,每条语句执行完毕后,系统都自动提交了.那么在基于行的复制中,可能会看到多条binlog 语句才对应一次commit,自然说明这是基于行的复制.

**还有一点,就是主库和从库的对应位置的event大小是相同的,例如上面的:**

25077342-25077069(从库上event大小)  =  863937234-863936961(主库上event大小)

否则,说明从库的relay-log丢失了,有可能是操作系统缓存丢失,也可能是mysql 异常crash导致relay-log buffer中的数据丢失.那么这时候就需要重设主从同步了.