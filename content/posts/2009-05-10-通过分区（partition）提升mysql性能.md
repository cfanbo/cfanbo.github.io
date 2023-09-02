---
title: 通过分区（Partition）提升MySQL性能
author: admin
type: post
date: 2009-05-10T13:29:00+00:00
excerpt: |
 什么是数据库分区？

 　　数据库分区是一种物理数据库设计技术，DBA和数据库建模人员对其相当熟悉。虽然分区技术可以实现很多效果，但其主要目的是为了在特定的SQL操作中减少数据读写的总量以缩减响应时间。
 分区主要有两种形式：//这里一定要注意行和列的概念（row是行，column是列）

 　　1. 水平分区（Horizontal Partitioning）这种形式分区是对表的行进行分区，通过这样的方式不同分组里面的物理列分割的数据集得以组合，从而进行个体分割（单分区）或集体分割（1个或多个分区）。所有在表中定义的列在每个数据集中都能找到，所以表的特性依然得以保持。
url: /archives/1374
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---
**什么是数据库分区？**

　　数据库分区是一种物理数据库设计技术，DBA和数据库建模人员对其相当熟悉。虽然分区技术可以实现很多效果，但其主要目的是为了在特定的SQL操作中减少数据读写的总量以缩减响应时间。
分区主要有两种形式：//这里一定要注意行和列的概念（row是行，column是列）

　　1. 水平分区（Horizontal Partitioning）这种形式分区是对表的行进行分区，通过这样的方式不同分组里面的物理列分割的数据集得以组合，从而进行个体分割（单分区）或集体分割（1个或多个分区）。所有在表中定义的列在每个数据集中都能找到，所以表的特性依然得以保持。
　　举个简单例子：一个包含十年发票记录的表可以被分区为十个不同的分区，每个分区包含的是其中一年的记录。（朋奕注：这里具体使用的分区方式我们后面再说，可以先说一点，一定要通过某个属性列来分割，譬如这里使用的列就是年份）
　　2. 垂直分区（Vertical Partitioning） 这种分区方式一般来说是通过对表的垂直划分来减少目标表的宽度，使某些特定的列 被划分到特定的分区，每个分区都包含了其中的列所对应的行。
　　举个简单例子：一个包含了大text和BLOB列的表，这些text和BLOB列又不经常被访问，这时候就要把这些不经常使用的text和BLOB了划分到另一个分区，在保证它们数据相关性的同时还能提高访问速度。

　　在数据库供应商开始在他们的数据库引擎中建立分区（主要是水平分区）时，DBA和建模者必须设计好表的物理分区结构，不要保存冗余的数据（不同表中同时都包含父表中的数据）或相互联结成一个逻辑父对象（通常是视图）。这种做法会使水平分区的大部分功能失效，有时候也会对垂直分区产生影响。

**在MySQL 5.1中进行分区**

　　MySQL5.1中最激动人心的新特性应该就是对水平分区的支持了。这对MySQL的使用者来说确实是个好消息，而且她已经支持分区大部分模式：
　　Range（范围） – 这种模式允许DBA将数据划分不同范围。例如DBA可以将一个表通过年份划分成三个分区，80年代（1980’s）的数据，90年代（1990’s）的数据以及任何在2000年（包括2000年）后的数据。
　　Hash（哈希） – 这中模式允许DBA通过对表的一个或多个列的Hash Key进行计算，最后通过这个Hash码不同数值对应的数据区域进行分区，。例如DBA可以建立一个对表主键进行分区的表。
　　Key（键值） – 上面Hash模式的一种延伸，这里的Hash Key是MySQL系统产生的。
　　List（预定义列表） – 这种模式允许系统通过DBA定义的列表的值所对应的行数据进行分割。例如：DBA建立了一个横跨三个分区的表，分别根据2004年2005年和2006年值所对应的数据。
　　Composite（复合模式） – 很神秘吧，哈哈，其实是以上模式的组合使用而已，就不解释了。举例：在初始化已经进行了Range范围分区的表上，我们可以对其中一个分区再进行hash哈希分区。

　　分区带来的好处太多太多了，有多少？俺也不知道，自己猜去吧，要是觉得没有多少就别用，反正俺也不求你用。不过在这里俺强调两点好处：

　　性能的提升（Increased performance） – 在扫描操作中，如果MySQL的优化器知道哪个分区中才包含特定查询中需要的数据，它就能直接去扫描那些分区的数据，而不用浪费很多时间扫描不需要的地方了。需要举个例子？好啊，百万行的表划分为10个分区，每个分区就包含十万行数据，那么查询分区需要的时间仅仅是全表扫描的十分之一了，很明显的对比。同时对十万行的表建立索引的速度也会比百万行的快得多得多。如果你能把这些分区建立在不同的磁盘上，这时候的I/O读写速度就“不堪设想”（没用错词，真的太快了，理论上100倍的速度提升啊，这是多么快的响应速度啊，所以有点不堪设想了）了。
对数据管理的简化（Simplified data management） – 分区技术可以让DBA对数据的管理能力提升。通过优良的分区，DBA可以简化特定数据操作的执行方式。例如：DBA在对某些分区的内容进行删除的同时能保证余下的分区的数据完整性(这是跟对表的数据删除这种大动作做比较的)。
此外分区是由MySQL系统直接管理的，DBA不需要手工的去划分和维护。例如：这个例如没意思，不讲了，如果你是DBA，只要你划分了分区，以后你就不用管了就是了。

　　站在性能设计的观点上，俺们对以上的内容也是相当感兴趣滴。通过使用分区和对不同的SQL操作的匹配设计，数据库的性能一定能获得巨大提升。下面咱们一起用用这个MySQL 5.1的新功能看看。
下面所有的测试都在Dell Optiplex box with a Pentium 4 3.00GHz processor, 1GB of RAM机器上（炫耀啊……），Fedora Core 4和MySQL 5.1.6 alpha上运行通过。

**如何进行实际分区**

　　看看分区的实际效果吧。我们建立几个同样的MyISAM引擎的表，包含日期敏感的数据，但只对其中一个分区。分区的表（表名为part_tab）我们采用Range范围分区模式，通过年份进行分区：

01. mysql> CREATE TABLE part_tab

02.    -> ( c1 int default NULL,

03.    -> c2 varchar(30) default NULL,

04.    -> c3 date default NULL

05.    ->

06.    -> ) engine=myisam

07. **-> PARTITION BY RANGE (year(c3)) (PARTITION p0 VALUES LESS THAN (1995),**
08. **-> PARTITION p1 VALUES LESS THAN (1996) , PARTITION p2 VALUES LESS THAN (1997) ,**
09. **-> PARTITION p3 VALUES LESS THAN (1998) , PARTITION p4 VALUES LESS THAN (1999) ,**
10. **-> PARTITION p5 VALUES LESS THAN (2000) , PARTITION p6 VALUES LESS THAN (2001) ,**
11. **-> PARTITION p7 VALUES LESS THAN (2002) , PARTITION p8 VALUES LESS THAN (2003) ,**
12. **-> PARTITION p9 VALUES LESS THAN (2004) , PARTITION p10 VALUES LESS THAN (2010),**
13. **-> PARTITION p11 VALUES LESS THAN MAXVALUE );**
14. Query OK, 0 rows affected (0.00 sec)


　　注意到了这里的最后一行吗？这里把不属于前面年度划分的年份范围都包含了，这样才能保证数据不会出错，大家以后要记住啊，不然数据库无缘无故出错你就爽了。那下面我们建立没有分区的表（表名为no\_part\_tab）：

1. mysql> create table no_part_tab

2.    -> (c1 int(11) default NULL,

3.    -> c2 varchar(30) default NULL,

4.    -> c3 date default NULL) engine=myisam;

5. Query OK, 0 rows affected (0.02 sec)


　　下面咱写一个存储过程（感谢Peter Gulutzan给的代码，如果大家需要Peter Gulutzan的存储过程教程的中文翻译也可以跟我要，chenpengyi◎gmail.com），它能向咱刚才建立的已分区的表中平均的向每个分区插入共8百万条不同的数据。填满后，咱就给没分区的克隆表中插入相同的数据：

01. mysql> delimiter //

02. mysql> CREATE PROCEDURE load_part_tab()

03.    -> begin

04.    -> declare v int default 0;

05.    -> while v < 8000000

06.    -> do

07.    -> insert into part_tab

08.    -> values (v,’testing partitions’,adddate(‘1995-01-01’,(rand(v)*36520) mod 3652));

09.    -> set v = v + 1;

10.    -> end while;

11.    -> end

12.    -> //

13. Query OK, 0 rows affected (0.00 sec)

14. mysql> delimiter ;

15. mysql> call load_part_tab();

16. Query OK, 1 row affected (8 min 17.75 sec)

17. mysql> insert into no_part_tab select * from part_tab;

18. Query OK, 8000000 rows affected (51.59 sec)

19. Records: 8000000 Duplicates: 0 Warnings: 0


　　表都准备好了。咱开始对这两表中的数据进行简单的范围查询吧。先分区了的，后没分区的，跟着有执行过程解析（MySQL Explain命令解析器），可以看到MySQL做了什么：

01. mysql> select count(*) from no_part_tab where

02.    -> c3 > date ‘1995-01-01’ and c3 < date ‘1995-12-31’;

03. +———-+

04. | count(*) |

05. +———-+

06. | 795181 |

07. +———-+

08. 1 row in set (38.30 sec)

09. mysql> select count(*) from part_tab where

10.    -> c3 > date ‘1995-01-01’ and c3 < date ‘1995-12-31’;

11. +———-+

12. | count(*) |

13. +———-+

14. | 795181 |

15. +———-+

16. 1 row in set (3.88 sec)

17. mysql> explain select count(*) from no_part_tab where

18.    -> c3 > date ‘1995-01-01’ and c3 < date ‘1995-12-31’\G

19. *************************** 1. row ***************************

20.    id: 1

21. select_type: SIMPLE

22.    table: no_part_tab

23.    type: ALL

24. possible_keys: NULL

25.    key: NULL

26.    key_len: NULL

27.    ref: NULL

28.    rows: 8000000

29.    Extra: Using where

30. 1 row in set (0.00 sec)

31. mysql> explain partitions select count(*) from part_tab where

32.    -> c3 > date ‘1995-01-01’ and c3 < date ‘1995-12-31’\G

33. *************************** 1. row ***************************

34.    id: 1

35. select_type: SIMPLE

36.    table: part_tab

37.    partitions: p1

38.    type: ALL

39. possible_keys: NULL

40.    key: NULL

41.    key_len: NULL

42.    ref: NULL

43.    rows: 798458

44.    Extra: Using where

45. 1 row in set (0.00 sec)


　　从上面结果可以容易看出，设计恰当表分区能比非分区的减少90％的响应时间。而命令解析Explain程序也告诉我们在对已分区的表的查询过程中仅对第一个分区进行了扫描，其他都跳过了。
哔厉吧拉，说阿说……反正就是这个分区功能对DBA很有用拉，特别对VLDB和需要快速反应的系统。

**对Vertical Partitioning的一些看法**
　　虽然MySQL 5.1自动实现了水平分区，但在设计数据库的时候不要轻视垂直分区。虽然要手工去实现垂直分区，但在特定场合下你会收益不少的。例如在前面建立的表中，VARCHAR字段是你平常很少引用的，那么对它进行垂直分区会不会提升速度呢？咱们看看测试结果：

01. mysql> desc part_tab;

02. +——-+————-+——+—–+———+——-+

03. | Field | Type | Null | Key | Default | Extra |

04. +——-+————-+——+—–+———+——-+

05. | c1 | int(11) | YES | | NULL | |

06. | c2 | varchar(30) | YES | | NULL | |

07. | c3 | date | YES | | NULL | |

08. +——-+————-+——+—–+———+——-+

09. 3 rows in set (0.03 sec)

10. mysql> alter table part_tab drop column c2;

11. Query OK, 8000000 rows affected (42.20 sec)

12. Records: 8000000 Duplicates: 0 Warnings: 0

13. mysql> desc part_tab;

14. +——-+———+——+—–+———+——-+

15. | Field | Type | Null | Key | Default | Extra |

16. +——-+———+——+—–+———+——-+

17. | c1 | int(11) | YES | | NULL | |

18. | c3 | date | YES | | NULL | |

19. +——-+———+——+—–+———+——-+

20. 2 rows in set (0.00 sec)

21. mysql> select count(*) from part_tab where

22.    -> c3 > date ‘1995-01-01’ and c3 < date ‘1995-12-31’;

23. +———-+

24. | count(*) |

25. +———-+

26. | 795181 |

27. +———-+

28. 1 row in set (0.34 sec)


　　在设计上去掉了VARCHAR字段后，不止是你，俺也发现查询响应速度上获得了另一个90％的时间节省。所以大家在设计表的时候，一定要考虑，表中的字段是否真正关联，又是否在你的查询中有用？

**补充说明**

　　这么简单的文章肯定不能说全MySQL 5.1 分区机制的所有好处和要点（虽然对自己写文章水平很有信心），下面就说几个感兴趣的：

　　* 支持所有存储引擎(MyISAM, Archive, InnoDB, 等等)
　　* 对分区的表支持索引，包括本地索引local indexes，对其进行的是一对一的视图镜像，假设一个表有十个分区，那么其本地索引也包含十个分区。
　　* 关于分区的元数据Metadata的表可以在INFORMATION_SCHEMA数据库中找到，表名为PARTITIONS。
　　* All SHOW 命令支持返回分区表以及元数据的索引。
　　* 对其操作的命令和实现的维护功能有（比对全表的操作还多）：

1. 　　o ADD PARTITION

2. 　　o DROP PARTITION

3. 　　o COALESCE PARTITION

4. 　　o REORGANIZE PARTITION

5. 　　o ANALYZE PARTITION

6. 　　o CHECK PARTITION

7. 　　o OPTIMIZE PARTITION

8. 　　o REBUILD PARTITION

9. 　　o REPAIR PARTITION


　　站在性能主导的观点上来说，MySQL 5.1的分区功能能给数据性能带来巨大的提升的同时减轻DBA的管理负担，如果分区合理的话。如果需要更多的资料可以去http://dev.mysql.com/doc/refman/5.1/en/partitioning.html或 http://forums.mysql.com/list.php?106获得相关资料。