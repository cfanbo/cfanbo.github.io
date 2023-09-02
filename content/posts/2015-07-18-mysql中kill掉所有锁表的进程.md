---
title: mysql中kill掉所有锁表的进程
author: admin
type: post
date: 2015-07-18T15:06:28+00:00
url: /archives/15869
categories:
 - MySQL
tags:
 - mysql

---
很多时候由于异常或程序错误会导致个别进程占用大量系统资源，需要结束这些进程，通常可以使用以下命令Kill进程:

mysql中kill掉所有锁表的进程

3点钟刚睡下, 4点多, 同事打电话告诉我用户数据库挂掉了. 我起床看一下进程列表.


```
mysql>show process list;
```

出来哗啦啦好几屏幕的, 没有一千也有几百条, 查询语句把表锁住了, 赶紧找出第一个Locked的thread_id, 在mysql的shell里面执行.


```
mysql>kill thread_id;
```

kill掉第一个锁表的进程, 依然没有改善. 既然不改善, 咱们就想办法将所有锁表的进程kill掉吧, 简单的脚本如下.


```
#!/bin/bash
mysql -u root -e "show processlist" | grep -i "Locked" >> locked_log.txt for line in `cat locked_log.txt | awk '{print $1}'`
do
echo "kill $line;" >> kill_thread_id.sql
done
```

现在kill_thread_id.sql的内容像这个样子


```
kill 66402982;
kill 66402983;
kill 66402986;
kill 66402991;
.....
```

好了, 我们在mysql的shell中执行, 就可以把所有锁表的进程杀死了.


```
mysql>source kill_thread_id.sql
```

当然了, 也可以一行搞定


```
for id in `mysqladmin processlist | grep -i locked | awk '{print $1}'`
do
mysqladmin kill ${id}
done
```

如果要查看阻塞语句的话，只需要将上面的”Locked”修改成”Sending”即可。

转自：