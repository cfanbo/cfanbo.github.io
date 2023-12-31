---
title: MongoDB的ObjectId
author: admin
type: post
date: 2011-07-27T02:31:18+00:00
url: /archives/10678
IM_contentdowned:
 - 1
categories:
 - nosql
tags:
 - MongoDB

---

前段时间有个朋友问我，分布式主键生成策略在我们这边是怎么实现的，当时我给的答案是sequence，当然这在不高并发的情况下是没有任何问题，实际上，我们的主键生成是可控的，但如果是在分布式高并发的情况下，那肯定是有问题的。

突然想起mongodb的objectid，记得以前看过文档，objectid是一种轻量型的，不同的机器都能用全局唯一的同种方法轻量的生成它，而不是采用传统的自增的主键策略，因为在多台服务器上同步自动增加主键既费力又费时，不得不佩服，mongodb从开始设计就被定义为分布式数据库。

下面深入一点来翻翻这个Objectid的底细，在mongodb集合中的每个document中都必须有一个”_id”建，这个键的值可以是任何类型的，在默认的情况下是个Objectid对象。

当我们让一个collection中插入一条不带_id的记录，系统会自动地生成一个_id的key


>

> > db.t_test.insert({“name”:”cyz”})

> db.t_test.findOne({“name”:”cyz”})

{ “_id” : ObjectId(“4df2dcec2cdcd20936a8b817”), “name” : “cyz” }
>

可以发现这里多出一个Objectid类型的_id，当然了，这个_id是系统默认生成的，你也可以为其指定一个值，不过在同一collections中该值必须是唯一的

把 ObjectId(“4df2dcec2cdcd20936a8b817”)这串值拿出来并对照官网的解析来深入分析。

“4df2dcec2cdcd20936a8b817” 以这段字符串为例来进行解析，这是一个24位的字符串，看起来很长，很难理解，实际上它是由ObjectId(string)所创建的一组十六进制的字符，每个字节两位的十六进制数字，总共使用了12字节的存储空间，可能有些朋友会感到很奇怪，居然是用了12个字节，而mysql的INT类型也只有4个字节，不过按照现在的存储设备，多出来的这点字节也应该不会成为什么瓶颈，实际上，mongodb在设计上无处不在的体现着用空间换时间的思想，接下看吧


下面是官网指定Bson中ObjectId的详细规范


[![](http://blog.haohtml.com/wp-content/uploads/2011/07/mongodb-objectid.png)](http://blog.haohtml.com/wp-content/uploads/2011/07/mongodb-objectid.png)

**TimeStamp**

前4位是一个unix的时间戳，是一个int类别，我们将上面的例子中的objectid的前4位进行提取“4df2dcec”，然后再将他们安装十六进制专为十进制：“1307761900”，这个数字就是一个时间戳，为了让效果更佳明显，我们将这个时间戳转换成我们习惯的时间格式


$ date -d ‘1970-01-01 UTC 1307761900  sec’  -u

2011年 06月 11日 星期六 03:11:40 UTC


前4个字节其实隐藏了文档创建的时间，并且时间戳处在于字符的最前面，这就意味着ObjectId大致会按照插入进行排序，这对于某些方面起到很大作用，如作为索引提高搜索效率等等。使用时间戳还有一个好处是，某些客户端驱动可以通过ObjectId解析出该记录是何时插入的，这也解答了我们平时快速连续创建多个Objectid时，会发现前几位数字很少发现变化的现实，因为使用的是当前时间，很多用户担心要对服务器进行时间同步，其实这个时间戳的真实值并不重要，只要其总不停增加就好。


**Machine**

接下来的三个字节，就是 2cdcd2 ,这三个字节是所在主机的唯一标识符，一般是机器主机名的散列值，这样就确保了不同主机生成不同的机器hash值，确保在分布式中不造成冲突，这也就是在同一台机器生成的objectid中间的字符串都是一模一样的原因。

**pid**

上面的Machine是为了确保在不同机器产生的objectid不冲突，而pid就是为了在同一台机器不同的mongodb进程产生了objectid不冲突，接下来的0936两位就是产生objectid的进程标识符。


**increment**

前面的九个字节是保证了一秒内不同机器不同进程生成objectid不冲突，这后面的三个字节a8b817，是一个自动增加的计数器，用来确保在同一秒内产生的objectid也不会发现冲突，允许256的3次方等于16777216条记录的唯一性。


**客户端生成**

mongodb产生objectid还有一个更大的优势，就是mongodb可以通过自身的服务来产生objectid，也可以通过客户端的驱动程序来产生，如果你仔细看文档你会感叹，mongodb的设计无处不在的使

用空间换时间的思想，比较objectid是轻量级，但服务端产生也必须开销时间，所以能从服务器转移到客户端驱动程序完成的就尽量的转移，必须将事务扔给客户端来完成，减低服务端的开销，另还有一点原因就是扩展应用层比扩展数据库层要变量得多。