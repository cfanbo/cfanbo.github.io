---
title: Cassandra数据模型
author: admin
type: post
date: 2010-08-14T13:45:16+00:00
url: /archives/5075
IM_contentdowned:
 - 1
categories:
 - 数据库
tags:
 - nosql

---
提起NoSQL这个话题，仿佛不应该是DBA要关注的事，而是架构师应该关心的。但是作为一名DBA，在使用传统的关系型思想建模时，应该有必要了解NoSQL的建模方法。

各种NoSQL数据库有很多，我最关注的还是**BigTable**类型，因为它是一个高可用可扩展的分布式计算平台，用来处理海量的结构化数据，而数据库同样也是处理结构化数据，所以除了没有SQL，在数据模型方面有相似之处。**Cassandra**是facebook开源出来的一个版本，可以认为是BigTable的一个开源版本，目前twitter和digg.com在使用。我们尝试从DBA的角度出发去理解Cassandra的数据模型。

NoSQL并不能简单的理解为**No SQL**，其本质应该是**No Relational**，也就是说它不是基于关系型的理论基础，而我们所有传统的数据库都是基于这套理论而发展起来的，所以SQL并不是问题的关键所在，比如有些NoSQL数据库可以提供SQL类型的接口，允许你通过类SQL的语法去访问数据。而Friendfeed则是反其道而行之，利用关系型数据库MySQL，采用了去关系化的设计方法，去实现自己的KeyValue存储。所以NoSQL的本质是No Relational.

**Cassandra特点：**

1.灵活的schema，不需要象数据库一样预先设计schema，增加或者删除字段非常方便（on the fly）。

2.支持range查询：可以对Key进行范围查询。

3.高可用，可扩展：单点故障不影响集群服务，可线性扩展。

**Keyspace**

Cassandra中的最大组织单元，里面包含了一系列Column family，Keyspace一般是应用程序的名称。你可以把它理解为Oracle里面的一个schema，包含了一系列的对象。

**Column family（CF）**

CF是某个特定Key的数据集合，每个CF物理上被存放在单独的文件中。从概念上看，CF有点象数据库中的Table.

**Key**

数据必须通过Key来访问，Cassandra允许范围查询，例如：`start => '10050', :finish => '10070'

`

**Column**

在Cassandra中字段是最小的数据单元，column和value构成一个对，比如：name:“jacky”，column是name，value是jacky，每个column:value后都有一个时间戳：timestamp。

和数据库不同的是，Cassandra的一行中可以有任意多个column，而且每行的column可以是不同的。从数据库设计的角度，你可以理解为表上有两个字段，第一个是Key，第二个是长文本类型，用来存放很多的column。这也是为什么说Cassandra具备非常灵活schema的原因。

**Super column
**

Super column是一种特殊的column，里面可以存放任意多个普通的column。而且一个CF中同样可以有任意多个Super column，一个CF只能定义使用Column或者Super column，不能混用。下面是Super column的一个例子，homeAddress这个Super column有三个字段：分别是street，city和zip：

```
homeAddress: {street: "binjiang road",city: "hangzhou",zip: "310052",}
```

**Sorting**

不同于数据库可以通过Order by定义排序规则，Cassandra取出的数据顺序是总是一定的，数据保存时已经按照定义的规则存放，所以取出来的顺序已经确定了，这是一个巨大的性能优势。有意思的是，**Cassandra按照column name而不是column value来进行排序，**它定义了以下几种选项：BytesType, UTF8Type, LexicalUUIDType, TimeUUIDType, AsciiType,  和LongType，用来定义如何按照column name来排序。实际上，就是把column name识别成为不同的类型，以此来达到灵活排序的目的。UTF8Type是把column name转换为UTF8编码来进行排序，LongType转换成为64位long型，TimeUUIDType是按照基于时间的UUID来排序。例如：

Column name按照LongType排序：

```
{name: 3, value: "jacky"},
{name: 123, value: "hellodba"},
{name: 976, value: "Cassandra"},
{name: 832416, value: "bigtable"}
```

Column name按照UTF8Type排序：

```
{name: 123, value: "hellodba"},
{name: 3, value: "jacky"},
{name: 832416, value: "bigtable"}
{name: 976, value: "Cassandra"}
```

下面我们看twitter的Schema：

```
<Keyspace Name="Twitter">
  <ColumnFamily CompareWith="UTF8Type" Name="Statuses" />
  <ColumnFamily CompareWith="UTF8Type" Name="StatusAudits" />
  <ColumnFamily CompareWith="UTF8Type" Name="StatusRelationships"
    CompareSubcolumnsWith="TimeUUIDType" ColumnType="Super" />
  <ColumnFamily CompareWith="UTF8Type" Name="Users" />
  <ColumnFamily CompareWith="UTF8Type" Name="UserRelationships"
    CompareSubcolumnsWith="TimeUUIDType" ColumnType="Super" />
</Keyspace>
```

我们看到一个叫Twitter的keyspace，包含若干个CF，其中StatusRelationships和UserRelationships被定义为包含Super column的CF，CompareWith定义了column的排序规则，CompareSubcolumnsWith定义了subcolumn的排序规则，这里使用了两种：TimeUUIDType和UTF8Type。我们没有看到任何有关column的定义，这意味着column是可以灵活变更的。

为了方便大家理解，我会尝试着用关系型数据库的建模方法去描述Twitter的Schema，但**千万不要误认为这就是Cassandra的数据模型**，对于Cassandra来说，每一行的colunn都可以是任意的，而不是象数据库一样需要在建表时就创建好。

[![](http://blog.haohtml.com/wp-content/uploads/2010/08/twitter.png)][1]

Users CF记录用户的信息，Statuses CF记录tweets的内容，StatusRelationships CF记录用户看到的tweets，UserRelationships CF记录用户看到的followers。我们注意到排序方式是TimeUUIDType，这个类型是按照时间进行排序的UUID字段，column name是用UUID函数产生（这个函数返回了一个UUID，这个UUID反映了当前的时间，可以根据这个UUID来排序，有点类似于timestamp一样），所以得到结果是按照时间来排序的。使用过twitter的人都知道，你总是可以看到自己最新的tweets或者最新的friends.

**存储**

Cassandra是基于列存储的(Bigtable也是一样)，这个和基于列的数据库是一个道理。

**[![](http://blog.haohtml.com/wp-content/uploads/2010/08/cassandra_data_model.png)][2]**

**API**

下面是数据库，Bigtable和Cassandra API的对比：

```
Relational			SELECT `column` FROM `database`.`table` WHERE `id` = key;
BigTable			table.get(key, "column_family:column")
Cassandra: standard model	keyspace.get("column_family", key, "column")
Cassandra: super column model	keyspace.get("column_family", key, "super_column", "column")
```

**我对Cassandra数据模型的理解：**

1.column name存放真正的值，而value是空。因为Cassandra是按照column name排序，而且是按列存储的，所以往往利用column name存放真正的值，而value部分则是空。例如：“jacky”:“null”，“fenng”:”null”

2.Super column可以看作是一个索引，有点象关系型数据库中的外键，利用super column可以实现快速定位，因为它可以返回一堆column，而且是排好序的。

3.排序在定义时就确定了，取出的数据肯定是按照确定的顺序排列的，这是一个巨大的性能优势。

4. 非常灵活的schema，column可以灵活定义。实际上，colume name在很多情况下，就是value（是不是有点绕）。

5.每个column后面的timestamp，我并没有找到明确的说明，我猜测可能是数据多版本，或者是底层清理数据时需要的信息。

最后说说架构，我认为架构的核心就是**有所取舍**，不管是**CAP**还是**BASE**，讲的都是这个原则。架构之美在于没有任何一种架构可以完美的解决各种问题，数据库和NoSQL都有其应用场景，我们要做的就是为自己找到合适的架构。

–EOF–

这篇文章，我参考了[up and running with cassandra][3]，除此以外，我还参考了twitter提供的API，它帮助我理解twitter的schema设计。这篇文章，肯定有很多理解不正确的地方，希望朋友们指正。

 [1]: http://blog.haohtml.com/wp-content/uploads/2010/08/twitter.png
 [2]: http://blog.haohtml.com/wp-content/uploads/2010/08/cassandra_data_model.png
 [3]: http://blog.evanweaver.com/articles/2009/07/06/up-and-running-with-cassandra/