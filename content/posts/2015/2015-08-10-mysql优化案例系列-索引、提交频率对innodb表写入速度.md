---
title: '[MySQL优化案例]系列 — 索引、提交频率对InnoDB表写入速度的影响'
author: admin
type: post
date: 2015-08-10T11:45:04+00:00
url: /archives/15948
categories:
 - MySQL
tags:
 - innodb
 - mysql优化

---
本次，我们通过对比，明明白白的知道索引、提交频率对InnoDB表写入速度的影响，了解有哪些需要注意的。

先直接说几个结论吧：

```
1、关于索引对写入速度的影响：
a、如果有自增列做主键，相对完全没索引的情况，写入速度约提升 3.11%；
b、如果有自增列做主键，并且二级索引，相对完全没索引的情况，写入速度约降低 27.37%；
```

因此，**InnoDB表最好总是有一个自增列做主键**。

2、关于提交频率对写入速度的影响（以表中只有自增列做主键的场景，一次写入数据30万行数据为例）：

```
a、等待全部数据写入完成后，最后再执行commit提交的效率最高；
b、每10万行提交一次，相对一次性提交，约慢了1.17%；
c、每1万行提交一次，相对一次性提交，约慢了3.01%；
d、每1千行提交一次，相对一次性提交，约慢了23.38%；
e、每100行提交一次，相对一次性提交，约慢了24.44%；
f、每10行提交一次，相对一次性提交，约慢了92.78%；
g、每行提交一次，相对一次性提交，约慢了546.78%，也就是慢了5倍；
```

因此，**最好是等待所有事务结束后再批量提交，而不是每执行完一个SQL就提交一次**。


曾经有一次对比测试mysqldump启用extended-insert和未启用导出的SQL脚本，**后者比前者慢了不止5倍**。

下面是详细的测试案例过程，有兴趣的同学可以看看：

```
DROP TABLE IF EXISTS \`mytab\`;
CREATE TABLE \`mytab\` (
\`id\` int(10) unsigned NOT NULL AUTO_INCREMENT,
\`c1\` int(11) NOT NULL DEFAULT ‘0’,
\`c2\` int(11) NOT NULL DEFAULT ‘0’,
\`c3\` timestamp NOT NULL DEFAULT CURRENT\_TIMESTAMP ON UPDATE CURRENT\_TIMESTAMP,
\`c4\` varchar(200) NOT NULL DEFAULT ”,
PRIMARY KEY (\`id\`)
) ENGINE=InnoDB;

DELIMITER $$$
DROP PROCEDURE IF EXISTS \`insert_mytab\`;

CREATE PROCEDURE \`insert_mytab\`(in rownum int, in commitrate int)
BEGIN
DECLARE i INT DEFAULT 0;

SET AUTOCOMMIT = 0;

WHILE i < rownum DO INSERT INTO mytab(c1, c2, c3,c4) VALUES( FLOOR(RAND()\*rownum),FLOOR(RAND()\*rownum),NOW(), REPEAT(CHAR(ROUND(RAND()\*255)),200)); SET i = i+1; /\* 达到每 COMMITRATE 频率时提交一次 */ IF (commitrate > 0) AND (i % commitrate = 0) THEN
COMMIT;
SELECT CONCAT(‘commitrate: ‘, commitrate, ‘ in ‘, I);
END IF;

END WHILE;

/\* 最终再提交一次,确保成功 \*/
COMMIT;
SELECT ‘ALL COMMIT;’;

END; $$$
```



#测试调用

```
call insert_mytab(300000, 1); — 每次一提交
call insert_mytab(300000, 10); — 每10次一提交
call insert_mytab(300000, 100); — 每100次一提交
call insert_mytab(300000, 1000); — 每1千次一提交
call insert_mytab(300000, 10000); — 每1万次提交
call insert_mytab(300000, 100000); — 每10万次一提交
call insert_mytab(300000, 0); — 一次性提交
```



测试耗时结果对比：
[![50810115001](https://blogstatic.haohtml.com//uploads/2023/09/50810115001.png)][1]

关于MySQL的方方面面大家想了解什么，可以直接留言回复，我会从中选择一些热门话题进行分享。 同时希望大家多多 **转发**，多一些阅读量是老叶继续努力分享的绝佳助力，谢谢大家 🙂

[1]: http://blog.haohtml.com/wp-content/uploads/2015/08/50810115001.png