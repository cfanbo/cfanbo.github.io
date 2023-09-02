---
title: 关于InnoDB表的page利用率和optimize table
author: admin
type: post
date: 2017-02-07T02:56:40+00:00
url: /archives/17400
categories:
 - MySQL

---
上一篇我们介绍了ibd_used这个工具，我们用来量化看表数据文件的page使用率。这里用来说明optimize table这个命令的问题和优化。

****实例准备****

建一个这样的表

 CREATE TABLE `tb` (

`seq_id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,


`a` varchar(32) DEFAULT NULL,


`b` varchar(32) DEFAULT NULL,


`c` varchar(32) DEFAULT NULL,


`d` char(255) DEFAULT NULL,


Primary key (seq_id),

KEY a (a),


KEY bc (b,c),


KEY cb (c,b)


) ENGINE=InnoDB DEFAULT CHARSET=utf8;

执行语句为“insert into tb(a,b,c) values(randstr, randstr, randstr);” randstr是客户端程序生成的长度30字节的随机字符串。30个线程并发，每个线程插入1w条记录。


等待更新完成后（包括purge完成，从系统的vmstat上看无任何io），执行./ibd_used tb.ibd 0 100000000，可以从最后4行看到各个索引的page平均利用率如下图。


[![](http://blog.haohtml.com/wp-content/uploads/2017/02/0b3726d7-f1a1-3f06-91a0-ba728b376bfa.jpg)](http://blog.haohtml.com/wp-content/uploads/2017/02/0b3726d7-f1a1-3f06-91a0-ba728b376bfa.jpg)

说明： 你会发现即使是主键索引，利用率也不一定很高。原因是什么？


**Optimize table** **效果**

我们知道Optimize table是用来作表整理的， 执行一下 optimize table tb，再看ibd_used的结果。

[![](http://blog.haohtml.com/wp-content/uploads/2017/02/29206cc5-f786-30d7-9e0f-f4e1167f0f7b.jpg)](http://blog.haohtml.com/wp-content/uploads/2017/02/29206cc5-f786-30d7-9e0f-f4e1167f0f7b.jpg)

说明：这里我们发现，pk的page利用率明显提升，是optimize效果，但是其他几个索引的page利用率却没有明显效果。为什么呢？


1)       首先是上面没有提的那个“异常”，既然是自增主键，为什么在optimize之前，pk的利用率不高？原因是多线程插入，虽然seq_id是递增申请，但不能保证是递增更新到page上。而通过optimize后，等于是单线程重新整理了。


2)       为什么其他索引的page利用率没有提升，这个就涉及到optimize table的内部执行过程。如下：


1. a) 建一个临时表，表结构与tb相同

2. b) 按照tb主键顺序将tb数据一行行的插入到临时表中

3. c) 删掉tb，临时表重命名为tb


所以我们看到对于其他索引，插入的值仍然是随机的过程。


**改进的思路**

我们知道InnoDB在5.1的时候innodb_plugin里面就有fast index creatation了，上述过程如果改成如下：


1. a) 建一个临时表，表结构与tb相同

2. b) 删掉临时表的所有非聚簇索引

3. c) 按照tb主键顺序将tb数据一行行的插入到临时表中

4. d) 建立临时表的所有非聚簇索引

5. e) 删掉tb，临时表重命名为tb


这样在执行步骤d)时，每个非聚簇索引都是按照排序好方式构建，则能让所有的索引page都很“紧凑”。


**Percona** **版本的** **expand_fast_index_creation** **参数**

在Percona版本中新增了这个参数，默认值是OFF，需要配置文件设置ON或者通过set命令热修改。


当设置为ON时，则optimize table tb实现的就是上述我们说到的改进流程。从ibd_used看到执行结果看到的效果如下：


[![](http://blog.haohtml.com/wp-content/uploads/2017/02/054c1738-cd22-335f-b448-fd73146967bb.jpg)](http://blog.haohtml.com/wp-content/uploads/2017/02/054c1738-cd22-335f-b448-fd73146967bb.jpg)

**小结**

所以当你需要通过optimze table优化表空间，


若是使用percona版本则最好先打开expand_fast_index_creation;


若是官方版本，则建议自己写脚本建临时表，按照上述的过程a~e来执行，达到最优的效果。


http://dinglin.iteye.com/blog/1504157