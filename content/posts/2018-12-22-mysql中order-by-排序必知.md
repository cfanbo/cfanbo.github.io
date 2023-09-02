---
title: MySQL中order by 排序必知
author: admin
type: post
date: 2018-12-22T07:53:16+00:00
url: /archives/18635
categories:
 - MySQL
tags:
 - mysql

---
在开发过程时，我们经常会遇到 order by 排序操作，那么你知道什么时候MySQL才会进行排序操作，什么时候不需要时间排序操作？，下面我们就从一个很小的例子中了解一下排序场景。

表结构如下：

```
CREATE TABLE t (
   id int(11) unsigned NOT NULL AUTO_INCREMENT,
   city varchar(16) NOT NULL,
   name varchar(16) NOT NULL,
   age int(11) NOT NULL,
   addr varchar(128) DEFAULT NULL,
   PRIMARY KEY (id),
   KEY city (city) USING BTREE
 ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

这里只有一个索引city，我们执行一条SQL语句：

```
SELECT city, name  FROM t WHERE city='杭州' ORDER BY name LIMIT 1000;
```

通过Explain命令查看执行情况

- ![](https://blog.haohtml.com/wp-content/uploads/2018/12/explain-1024x119.jpg)



发现Extra字段里有”Using filesort”，说明使用了排序，而排序必用到了`sort_buffer`, 这是由数据库为了专门进行排序操作而分配的一块内存。
这里的Using index conditon是指ICP特性，请参考：

下面我们专门来看一下这条语句的执行流程是如何的？在此以前我们需要了解一个概念，就是索引的特性是有序的，系统搜索时，一旦找到最后一条满足条件的记录后就立即停止，后面的记录则不再进行扫描，这样就可以用来解决全表扫描的性能问题。

**这里执行过程大概如下：**

1、从city索引树中第一条记录开始扫描，找到第一条满足条件的记录，获取到主键ID（每个索引都包含主键ID）。如果第一条记录的city不是”杭州”的话，则继续下一条。
2、根据主键ID进行回表，将 city, name 读取出来放在 `sort_buffer`
3、继续下一条并重复1、2步骤
4、直到所有满足条件的记录都放到了`sort_buffer`后，根据name进行排序
5、读取前1000条记录，并返回给客户端

可以看到我们这里使用到了在`sort_buffer`中排序操作。

我们一般将这个排序过程称为 “全字段” 排序。

下面我们修改一个city索引，执行SQL语句

```
alter table t drop index city;
alter table t add index (city,name);
```

这里我们将city索引包含了city和name两个字段。现在我们再用Explain查看一下执行结果.![](https://blog.haohtml.com/wp-content/uploads/2018/12/explain_2-1024x144.jpg)

发现Extra字段变成了 “Using where; Using index”，原来的”Using filesort”消失了，多了一个 “Using index”。说明没有了排序操作，并且使用到了索引，这里使用的是覆盖索引。索引中已经包含了我们所需要的city和name字段，且**索引都是已经排序过的**，正好符合了我们的要求，即不需要进行回表操作，也不需要用到sort_buffer排序操作了。所以这里不需要用到排序操作。

**执行过程如下：**

1、搜索city索引树，找到第一条**索引记录**，直接返回索引中的city和name字段
2、继续下一条,重复上面的步骤
3、找到1000条记录的时候立即停止，直接返回给客户端就可以了（丢失后面的记录，不管它满足不满足，反正要的数据已经够了）

下面我们再来看一下另一个SQL语句是否会用到索引呢？

```
SELECT city,name FROM t WHERE city IN ('杭州', '苏州') ORDER BY name LIMIT 100;
```

这里仍是按name进行排序，但城市变成了2个，对于单个city 内部，name是递增的，但这条SQL语句是查询了两个城市，因此就不满足name递增的条件了，所以要想取到排序后的两个城市的结果就需要将两个城市所有记录都存储到sort buffer中，然后再进行排序操作，最后返回给客户端。 很明显这条语句会有排序排序。

如果想不使用排序实现上面的结果的话，则可以将上面的SQL语句分解成两个，然后在**客户端**使用归并算法进行排序。

```
SELECT city,name FROM t  WHERE city='杭州' ORDER BY name LIMIT 100;
```

和

```
SELECT city,name FROM t  WHERE city='苏州' ORDER BY name LIMIT 100;
```

从合并结果里取出前100条记录返回给客户端即可。

**执行过程如下：**

1、搜索city 索引树，找到满足条件的记录(city为杭州或者苏州），获取city和name字段
2、继续下一条，重复上面的步骤
3、直到所有的记录都放在了sort buffer中，根据name进行排序，返回给客户端

**最后：**

这里需要提醒一下大家要注意排序参数 sort\_buffer\_size 大小的设置，如果设置过小，而排序时系统读取的字段内容过多的话，可能需要借助临时磁盘文件排序（系统会根据排序数据量大小分成多个文件排序，然后使用并归排序算法再次排序，排序效率会慢很多，所以避免使用磁盘相关的交互）。如果设置的大小大于排序的内容的话，则直接在sort buffer内存就排序好了，内存效率要比磁盘高的太多了。

另分析SQL执行结果效率除了Explain以后，可以使用以下方法

/\* 查看 OPTIMIZER_TRACE 输出 \*/
SELECT * FROM `information_schema`.`OPTIMIZER_TRACE`

可以查看是否使用了临时文件，其中的number\_of\_tmp_files表示使用到了多少个临时文件，如果表示0则表示未使用到临时文件，如果大于0(如8)，则表示排序的时候将分成8个文件分别进行分组排序，最后再使用归并算法合到一起，最终返回排序的结果，返回给客户端。