---
title: MYSQL主从失败，报错 Got fatal error 1236 后恢复过程
author: admin
type: post
date: 2011-01-06T09:38:27+00:00
url: /archives/7455
IM_contentdowned:
 - 1
categories:
 - MySQL

---
环境：
Mysql: 5.1.37 dual master(节点为A,B)
OS: centos5.3 x64

由于我今天突然将重新启动从服务，导致MYSQL一边的复制失败，如下：

从服务器节点A启动slave就报下面的错误：

> 090910 22:47:18 [ERROR] Error reading packet from server: Client requested master to start replication from impossible position ( server_errno=1236)
> 090910 22:47:18 [ERROR] Got fatal error 1236: ‘Client requested master to start replication from impossible position’ from master when reading data from binary log
> 090910 22:47:18 [Note] Slave I/O thread exiting, read up to log ‘mysql-bin.000008’, position 753871857

主服务器节点B是正常的。

有时提示信息里没有最后一行的,这个时候可以通过查看slave状态里的 **Read_Master_Log_Pos** 的值看出来的．

先偿试手动change master:

> CHANGE MASTER TO MASTER_HOST=’xxxx.xxx.xxx.xxx’,
> -> MASTER_USER=’repl’,
> -> MASTER_PASSWORD=’xxxxx’,
> -> MASTER\_LOG\_FILE=’binlog.000008′,
> -> MASTER\_LOG\_POS=753871857;

但是问题依旧，仔细看错误的描述，里面说753871857这个position是impossible position。
难道说mysql-bin.000008里面没有这个位置的？

登陆到主服务器B节点上面用mysqlbinlog查看

> [root@im_offlog2b mysql]# mysqlbinlog –start-position=753871857 mysql-bin.000008
> /\*!40019 SET @@session.max\_insert\_delayed_threads=0\*/;
> /\*!50003 SET @OLD\_COMPLETION\_TYPE=@@COMPLETION\_TYPE,COMPLETION\_TYPE=0\*/;
> DELIMITER /\*!\*/;
> DELIMITER ;
> \# End of log file
> ROLLBACK /\* added by mysqlbinlog \*/;
> /\*!50003 SETCOMPLETION\_TYPE=@OLD\_COMPLETION_TYPE\*/;

好像是没有。上面的命令也可以加一个**–end-position**选项
为了进一步确认，我将binlog dump成文本文件

> mysqlbinlog mysql-bin.000008 > 1.txt
> tail -n 100000 1.txt > 2.txt

然后打开2.txt文件，跳到最后：

> \# at 753870260
> #080724 16:21:25 server id 2 end\_log\_pos 753870665 Query thread\_id=185 exec\_time=0 error_code=0
> SET TIMESTAMP=1216887685/\*!\*/;
> insert into im\_offlinemsg\_200807(gmt\_create,type,from\_id,to_id,content)values(sysdate(),0,’cnalichnzizufhm’,’cnalichnluelee’,’AAFcQzB4NzBmZlxGy87M5VxUob7Iureiob9cQzBcUzB4OS4weGI0XEbLzszlXFRodHRwOi8vYmJzLmticmVuLmNuL3RhaXdhbi5odG1sDQq7qMHLztLV+9X70rvN7cnPtcTKsbzksKGjrNbV09q447rDwcujrL/syKW/tL+0o6y8x7XDwfS49tHUxbZcVC86JFxULzpnaXJs’)/\*!\*/;
>
> \# at 753870665
> #080724 16:21:25 server id 2 end\_log\_pos 753870692 Xid = 35714167
> COMMIT/\*!\*/;
> DELIMITER ;
> \# End of log file
> ROLLBACK /\* added by mysqlbinlog \*/;
> /\*!50003 SETCOMPLETION\_TYPE=@OLD\_COMPLETION_TYPE\*/;

发现mysql-bin.000008里面最接近753871857的一个有效的position是753870665。好像有error_code是无效的,下面我们就可以用这个新position值来重新设定从的复制起始地址了。

首先在从服务器上停止复制状态.

> SLAVE STOP;

然后再change master，将位置指定为这个最后位置：

> HANGE MASTER TO
> ->MASTER_HOST=’172.18.57.154′,
> -> MASTER_USER=’hoahtml’,
> -> MASTER_PASSWORD=’haohtmlpassword’,
> -> MASTER\_LOG\_FILE=’mysql-bin.000008′,
> -> MASTER\_LOG\_POS=753870665;

重新启动后，再执行

> SLAVE START;

检查是否正常

> show slave status\G;

命令查看当前的状态,发现两步字.问题就解决了。

**参考文档:** [mysql集群从服务器复制传递和状态文件(master.info和relay_log.info)](../index.php/archives/3463)