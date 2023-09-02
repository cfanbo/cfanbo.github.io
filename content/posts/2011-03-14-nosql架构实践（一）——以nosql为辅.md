---
title: NoSQL架构实践（一）——以NoSQL为辅
author: admin
type: post
date: 2011-03-14T09:39:39+00:00
url: /archives/7984
IM_data:
 - 'a:3:{s:94:"http://www.infoq.com/resource/news/2011/02/nosql-architecture-practice/zh/resources/image1.JPG";s:66:"http://blog.haohtml.com/wp-content/uploads/2011/03/d64e_image1.JPG";s:94:"http://www.infoq.com/resource/news/2011/02/nosql-architecture-practice/zh/resources/image2.JPG";s:66:"http://blog.haohtml.com/wp-content/uploads/2011/03/3d50_image2.JPG";s:94:"http://www.infoq.com/resource/news/2011/02/nosql-architecture-practice/zh/resources/image3.JPG";s:66:"http://blog.haohtml.com/wp-content/uploads/2011/03/27f7_image3.JPG";}'
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql
 - nosql

---
本文为凤凰网负责底层组的研发工作的[孙立][1]同学在InfoQ上发表的NoSQL系列文章之一，本文探讨了使用NoSQL作存储的几种典型模式，对NoSQL的实践及应用，具有很强的指导意义。

> 前面[《为什么要使用NoSQL》][2]和[《关系数据库还是NoSQL数据库》][3]两篇从大体上介绍了为什么要用NoSQL，何时该用NoSQL。经常有朋友遇到困惑，看到NoSQL的介绍，觉得很好，但是却不知道如何正式用到自己的项目中。很大的原因就是思维固定在MySQL中了，他们问得最多的问题就是用了NoSQL，我如何做关系查询。那么接下来，我们看下怎么样在我们的系统中使用NoSQL。
>
> 怎么样把NoSQL引入到我们的系统架构设计中，需要根据我们系统的业务场景来分析，什么样类型的数据适合存储在NoSQL数据库中，什么样类型的数据必须使用关系数据库存储。明确引入的NoSQL数据库带给系统的作用，它能解决什么问题，以及可能带来的新的问题。下面我们分析几种常见的NoSQL架构。
>
> ## **（一）NoSQL作为镜像**
>
> 不改变原有的以MySQL作为存储的架构，使用NoSQL作为辅助镜像存储，用NoSQL的优势辅助提升性能。
>
>![](http://www.infoq.com/resource/news/2011/02/nosql-architecture-practice/zh/resources/image1.JPG)
>
> **图 1 -NoSQL为镜像（代码完成模式 ）**
>
>

>
>
>
>
>
>
>
>

```
1
2
3
4
5
6
7
8
```

>
>

```
//写入数据的示例伪代码
//data为我们要存储的数据对象
data.title=”title”;
data.name=”name”;
data.time=”2009-12-01 10:10:01”;
data.from=”1”;
id=DB.Insert(data);//写入MySQL数据库
NoSQL.Add(id,data);//以写入MySQL产生的自增id为主键写入NoSQL数据库
```

>
>

>
> 如果有数据一致性要求，可以像如下的方式使用
>
>

>
>
>
>
>
>
>
>

```
1
2
3
4
5
6
7
8
9
10
11
12
13
```

>
>

```
//写入数据的示例伪代码
//data为我们要存储的数据对象
bool status=false;
DB.startTransaction();//开始事务
id=DB.Insert(data);//写入MySQL数据库
if(id>0){
    status=NoSQL.Add(id,data);//以写入MySQL产生的自增id为主键写入NoSQL数据库
}
if(id>0 && status==true){
    DB.commit();//提交事务
}else{
    DB.rollback();//不成功，进行回滚
}
```

>
>

>
> 上面的代码看起来可能觉得有点麻烦，但是只需要在DB类或者ORM层做一个统一的封装，就能实现重用了，其他代码都不用做任何的修改。
>
> 这种架构在原有基于MySQL数据库的架构上增加了一层辅助的NoSQL存储，代码量不大，技术难度小，却在可扩展性和性能上起到了非常大的作用。只需要程序在写入MySQL数据库后，同时写入到NoSQL数据库，让MySQL和NoSQL拥有相同的镜像数据，在某些可以根据主键查询的地方，使用高效的NoSQL数据库查询，这样就节省了MySQL的查询，用NoSQL的高性能来抵挡这些查询。
>
>![](http://www.infoq.com/resource/news/2011/02/nosql-architecture-practice/zh/resources/image2.JPG)
>
> **图 2 -NoSQL为镜像（同步模式）**
>
> 这种不通过程序代码，而是通过MySQL把数据同步到NoSQL中，这种模式是上面一种的变体，是一种对写入透明但是具有更高技术难度一种模式。这种模式适用于现有的比较复杂的老系统，通过修改代码不易实现，可能引起新的问题。同时也适用于需要把数据同步到多种类型的存储中。
>
> MySQL到NoSQL同步的实现可以使用MySQL UDF函数，MySQL binlog的解析来实现。可以利用现有的开源项目来实现，比如：
>
> * [MySQL memcached UDFs][4]：从通过UDF操作Memcached协议。
> * 国内张宴开源的[mysql-udf-http][5]：通过UDF操作http协议。
>
> 有了这两个MySQL UDF函数库，我们就能通过MySQL透明的处理Memcached或者Http协议，这样只要有兼容Memcached或者Http协议的NoSQL数据库，那么我们就能通过MySQL去操作以进行同步数据。再结合[lib\_mysqludf\_json][6]，通过UDF和MySQL触发器功能的结合，就可以实现数据的自动同步。
>
> ## **（二）MySQL和NoSQL组合**
>
> MySQL中只存储需要查询的小字段，NoSQL存储所有数据。
>
>![](http://www.infoq.com/resource/news/2011/02/nosql-architecture-practice/zh/resources/image3.JPG)
>
> **图 3 -MySQL和NoSQL组合**
>
>

>
>
>
>
>
>
>
>

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
```

>
>

```
//写入数据的示例伪代码

//data为我们要存储的数据对象
data.title=”title”;
data.name=”name”;
data.time=”2009-12-01 10:10:01”;
data.from=”1”;
bool status=false;
DB.startTransaction();//开始事务
id=DB.Insert(“INSERT INTO table (from) VALUES(data.from)”);//写入MySQL数据库,只写from需要where查询的字段
if(id>0){
    status=NoSQL.Add(id,data);//以写入MySQL产生的自增id为主键写入NoSQL数据库
}
if(id>0 && status==true){
    DB.commit();//提交事务
}else{
    DB.rollback();//不成功，进行回滚
}
```

>
>

>
> 把需要查询的字段，一般都是数字，时间等类型的小字段存储于MySQL中，根据查询建立相应的索引，其他不需要的字段，包括大文本字段都存储在NoSQL中。在查询的时候，我们先从MySQL中查询出数据的主键，然后从NoSQL中直接取出对应的数据即可。
>
> 这种架构模式把MySQL和NoSQL的作用进行了融合，各司其职，让MySQL专门负责处理擅长的关系存储，NoSQL作为数据的存储。它有以下优点：
>
> * 节省MySQL的IO开销。由于MySQL只存储需要查询的小字段，不再负责存储大文本字段，这样就可以节省MySQL存储的空间开销，从而节省MySQL的磁盘IO。我们曾经通过这种优化，把MySQL一个40G的表缩减到几百M。
> * 提高MySQl Query Cache缓存命中率。我们知道query cache缓存失效是表级的，在MySQL表一旦被更新就会失效，经过这种字段的分离，更新的字段如果不是存储在MySQL中，那么对query cache就没有任何影响。而NoSQL的Cache往往都是行级别的，只对更新的记录的缓存失效。
> * 提升MySQL主从同步效率。由于MySQL存储空间的减小，同步的数据记录也减小了，而部分数据的更新落在NoSQL而不是MySQL，这样也减少了MySQL数据需要同步的次数。
> * 提高MySQL数据备份和恢复的速度。由于MySQL数据库存储的数据的减小，很容易看到数据备份和恢复的速度也将极大的提高。
> * 比以前更容易扩展。NoSQL天生就容易扩展。经过这种优化，MySQL性能也得到提高。
>
> 比如手机凤凰网就是这种架构 [http://www.cnblogs.com/sunli/archive/2010/12/20/imcp.html][7]
>
> ## **总结**
>
> 以NoSQL为辅的架构还是以MySQL架构的思想为中心，只是在以前的架构上辅助增加了NoSQL来提高其性能和可扩展性。这种架构实现起来比较容易，却能取得不错的效果。如果正想在项目中引入NoSQL，或者你的以MySQL架构的系统目前正出现相关的瓶颈，希望本文可以为你带来帮助。
>
> * * *
>
> 感谢[张凯峰][8]对本文的审校。

原文链接：



 [1]: http://t.sina.com.cn/sunli1223
 [2]: http://blog.nosqlfan.com/html/1087.html
 [3]: http://www.infoq.com/cn/news/2011/01/relation-db-nosql-db
 [4]: http://dev.mysql.com/doc/refman/5.0/en/ha-memcached-interfaces-mysqludf.html
 [5]: http://blog.s135.com/mysql-udf-http/
 [6]: http://www.mysqludf.org/lib_mysqludf_json/index.php
 [7]: http://www.google.com/url?q=http%3A%2F%2Fwww.cnblogs.com%2Fsunli%2Farchive%2F2010%2F12%2F20%2Fimcp.html&sa=D&sntz=1&usg=AFQjCNFPdKEh6vlSCW-j6e21G2CAfsZZTg
 [8]: http://www.infoq.com/cn/bycategory.action?authorName=%E5%BC%A0%E5%87%AF%E5%B3%B0