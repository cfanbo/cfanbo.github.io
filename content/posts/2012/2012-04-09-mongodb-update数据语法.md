---
title: MongoDB update数据语法
author: admin
type: post
date: 2012-04-09T10:20:26+00:00
url: /archives/12735
IM_contentdowned:
 - 1
categories:
 - nosql
tags:
 - MongoDB

---
在前面的文章“ [mongodb 查询的语法](http://hi.baidu.com/farmerluo/blog/item/2d15e95027aa86501138c27a.html)”里，我介绍了Mongodb的常用查询语法，Mongodb的update操作也有点复杂，我结合自己的使用经验，在这里介绍一下，给用mongodb的朋友看看，也方便以后自己用到的时候查阅：

注：在这篇文章及上篇文章内讲的语法介绍都是在mongodb shell环境内的，和真正运用语言编程（如java,php等）使用时，在使用方法上会有一些差别，但语法(如查询条件，$in,$inc等)是一样的。

本文是参考官方文档来介绍的，之所以有官方文档还要在这介绍，一方面是就当翻译，毕竟每次要用时去看英文文档比较累，第二是官方文档讲解比较简单，有时光看官方文档不好理解，我在实际操作的情况下可以做些补充。

好了，不多说了，下面正式开始：

mongodb更新有两个命令：

**1).update()命令**

db.collection.update( criteria, objNew, upsert, multi )

criteria : update的查询条件，类似sql update查询内where后面的
objNew   : update的对象和一些更新的操作符（如$,$inc…）等，也可以理解为sql update查询内set后面的
upsert   : 这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
multi    : mongodb默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。

例：
db.test0.update( { “count” : { $gt : 1 } } , { $set : { “test2” : “OK”} } ); 只更新了第一条记录
db.test0.update( { “count” : { $gt : 3 } } , { $set : { “test2” : “OK”} },false,true ); 全更新了
db.test0.update( { “count” : { $gt : 4 } } , { $set : { “test5” : “OK”} },true,false ); 只加进去了第一条
db.test0.update( { “count” : { $gt : 5 } } , { $set : { “test5” : “OK”} },true,true ); 全加进去了
db.test0.update( { “count” : { $gt : 15 } } , { $inc : { “count” : 1} },false,true );全更新了
db.test0.update( { “count” : { $gt : 10 } } , { $inc : { “count” : 1} },false,false );只更新了第一条

**2).save()命令**

db.collection.save( x )

x就是要更新的对象，只能是单条记录。

如果在collection内已经存在一个和x对象相同的”\_id”的记录。mongodb就会把x对象替换collection内已经存在的记录，否则将会插入x对象，如果x内没有\_id,系统会自动生成一个再插入。相当于上面update语句的upsert=true,multi=false的情况。

例：
db.test0.save({count:40,test1:”OK”}); #_id系统会生成
db.test0.save({\_id:40,count:40,test1:”OK”}); #如果test0内有\_id等于40的，会替换，否则插入。

mongodb的更新操作符：

**1) $inc**

用法：{ $inc : { field : value } }

意思对一个数字字段field增加value，例：

> db.test0.find( { “_id” : 15 } );
{ “_id” : { “floatApprox” : 15 }, “count” : 16, “test1” : “TESTTEST”, “test2” : “OK”, “test3” : “TESTTEST”, “test4” : “OK”, “test5” : “OK” }

> db.test0.update( { “_id” : 15 } , { $inc : { “count” : 1 } } );
> db.test0.find( { “_id” : 15 } );
{ “_id” : { “floatApprox” : 15 }, “count” : 17, “test1” : “TESTTEST”, “test2” : “OK”, “test3” : “TESTTEST”, “test4” : “OK”, “test5” : “OK” }

> db.test0.update( { “_id” : 15 } , { $inc : { “count” : 2 } } );
> db.test0.find( { “_id” : 15 } );
{ “_id” : { “floatApprox” : 15 }, “count” : 19, “test1” : “TESTTEST”, “test2” : “OK”, “test3” : “TESTTEST”, “test4” : “OK”, “test5” : “OK” }

> db.test0.update( { “_id” : 15 } , { $inc : { “count” : -1 } } );
> db.test0.find( { “_id” : 15 } );
{ “_id” : { “floatApprox” : 15 }, “count” : 18, “test1” : “TESTTEST”, “test2” : “OK”, “test3” : “TESTTEST”, “test4” : “OK”, “test5” : “OK” }

**2) $set**

用法：{ $set : { field : value } }

就是相当于sql的set field = value，全部数据类型都支持$set。例：
> db.test0.update( { “_id” : 15 } , { $set : { “test1” : “testv1″,”test2” : “testv2″,”test3” : “testv3″,”test4” : “testv4” } } );
> db.test0.find( { “_id” : 15 } );
{ “_id” : { “floatApprox” : 15 }, “count” : 18, “test1” : “testv1”, “test2” : “testv2”, “test3” : “testv3”, “test4” : “testv4”, “test5” : “OK” }

**3) $unset**

用法：{ $unset : { field : 1} }

顾名思义，就是删除字段了。例：
> db.test0.update( { “_id” : 15 } , { $unset : { “test1”:1 } } );
> db.test0.find( { “_id” : 15 } );
{ “_id” : { “floatApprox” : 15 }, “count” : 18, “test2” : “testv2”, “test3” : “testv3”, “test4” : “testv4”, “test5” : “OK” }

> db.test0.update( { “_id” : 15 } , { $unset : { “test2”: 0 } } );
> db.test0.find( { “_id” : 15 } );
{ “_id” : { “floatApprox” : 15 }, “count” : 18, “test3” : “testv3”, “test4” : “testv4”, “test5” : “OK” }

> db.test0.update( { “_id” : 15 } , { $unset : { “test3”:asdfasf } } );
Fri May 14 16:17:38 JS Error: ReferenceError: asdfasf is not defined (shell):0

> db.test0.update( { “_id” : 15 } , { $unset : { “test3″:”test” } } );
> db.test0.find( { “_id” : 15 } );
{ “_id” : { “floatApprox” : 15 }, “count” : 18, “test4” : “testv4”, “test5” : “OK” }

没看出field : 1里面的1是干什么用的，反正只要有东西就行。

**4) $push**

```
用法：{ $push : { field : value } }

把value追加到field里面去，field一定要是数组类型才行，如果field不存在，会新增一个数组类型加进去。例：

> db.test0.update( { "_id" : 15 } , { $set : { "test1" : ["aaa","bbb"] } } );
> db.test0.find( { "_id" : 15 } );
{ "_id" : { "floatApprox" : 15 }, "count" : 18, "test1" : [ "aaa", "bbb" ], "test4" : "testv4", "test5" : "OK" }

> db.test0.update( { "_id" : 15 } , { $push : { "test1": "ccc" } } );
> db.test0.find( { "_id" : 15 } );
{ "_id" : { "floatApprox" : 15 }, "count" : 18, "test1" : [ "aaa", "bbb", "ccc" ], "test4" : "testv4", "test5" : "OK" }

> db.test0.update( { "_id" : 15 } , { $push : { "test2": "ccc" } } );
> db.test0.find( { "_id" : 15 } );
{ "_id" : { "floatApprox" : 15 }, "count" : 18, "test1" : [ "aaa", "bbb", "ccc" ], "test2" : [ "ccc" ], "test4" : "testv4", "test5" : "OK" }

> db.test0.update( { "_id" : 15 } , { $push : { "test1": ["ddd","eee"] } } );
> db.test0.find( { "_id" : 15 } );
{ "_id" : { "floatApprox" : 15 }, "count" : 18, "test1" : [ "aaa", "bbb", "ccc", [ "ddd", "eee" ] ], "test2" : [ "ccc" ], "test4" : "testv4", "test5" : "OK" }
```

**5) $pushAll**

用法：{ $pushAll : { field : value_array } }

同$push,只是一次可以追加多个值到一个数组字段内。例：

> db.test0.find( { “_id” : 15 } );
{ “_id” : { “floatApprox” : 15 }, “count” : 18, “test1” : [ “aaa”, “bbb”, “ccc”, [ “ddd”, “eee” ] ], “test2” : [ “ccc” ], “test4” : “testv4”, “test5” : “OK” }

> db.test0.update( { “_id” : 15 } , { $pushAll : { “test1”: [“fff”,”ggg”] } } );
> db.test0.find( { “_id” : 15 } );
{ “_id” : { “floatApprox” : 15 }, “count” : 18, “test1” : [ “aaa”, “bbb”, “ccc”, [ “ddd”, “eee” ], “fff”, “ggg” ], “test2” : [ “ccc” ], “test4” : “testv4”, “test5” : “OK” }

**6)  $addToSet**

```
用法：{ $addToSet : { field : value } }

增加一个值到数组内，而且只有当这个值不在数组内才增加。例：
> db.test0.update( { "_id" : 15 } , { $addToSet : { "test1": {$each : ["444","555"] } } } );
> db.test0.find( { "_id" : 15 } );
{ "_id" : { "floatApprox" : 15 }, "count" : 18, "test1" : [
        "aaa",
        "bbb",
        "ccc",
        [
                "ddd",
                "eee"
        ],
        "fff",
        "ggg",
        [
                "111",
                "222"
        ],
        "444",
        "555"
], "test2" : [ "ccc" ], "test4" : "testv4", "test5" : "OK" }
> db.test0.update( { "_id" : 15 } , { $addToSet : { "test1": {$each : ["444","555"] } } } );
> db.test0.find( { "_id" : 15 } );
{ "_id" : { "floatApprox" : 15 }, "count" : 18, "test1" : [
        "aaa",
        "bbb",
        "ccc",
        [
                "ddd",
                "eee"
        ],
        "fff",
        "ggg",
        [
                "111",
                "222"
        ],
        "444",
        "555"
], "test2" : [ "ccc" ], "test4" : "testv4", "test5" : "OK" }
> db.test0.update( { "_id" : 15 } , { $addToSet : { "test1": ["444","555"]  } } );
> db.test0.find( { "_id" : 15 } );
{ "_id" : { "floatApprox" : 15 }, "count" : 18, "test1" : [
        "aaa",
        "bbb",
        "ccc",
        [
                "ddd",
                "eee"
        ],
        "fff",
        "ggg",
        [
                "111",
                "222"
        ],
        "444",
        "555",
        [
                "444",
                "555"
        ]
], "test2" : [ "ccc" ], "test4" : "testv4", "test5" : "OK" }
> db.test0.update( { "_id" : 15 } , { $addToSet : { "test1": ["444","555"]  } } );
> db.test0.find( { "_id" : 15 } );
{ "_id" : { "floatApprox" : 15 }, "count" : 18, "test1" : [
        "aaa",
        "bbb",
        "ccc",
        [
                "ddd",
                "eee"
        ],
        "fff",
        "ggg",
        [
                "111",
                "222"
        ],
        "444",
        "555",
        [
                "444",
                "555"
        ]
], "test2" : [ "ccc" ], "test4" : "testv4", "test5" : "OK" }

7) $pop

删除数组内的一个值

用法：
删除最后一个值：{ $pop : { field : 1  } }
```

```
删除第一个值：{ $pop : { field : -1  } }

注意，只能删除一个值，也就是说只能用1或-1，而不能用2或-2来删除两条。mongodb 1.1及以后的版本才可以用，例：
> db.test0.find( { "_id" : 15 } );
{ "_id" : { "floatApprox" : 15 }, "count" : 18, "test1" : [
        "bbb",
        "ccc",
        [
                "ddd",
                "eee"
        ],
        "fff",
        "ggg",
        [
                "111",
                "222"
        ],
        "444"
], "test2" : [ "ccc" ], "test4" : "testv4", "test5" : "OK" }
> db.test0.update( { "_id" : 15 } , { $pop : { "test1": -1 } } );
> db.test0.find( { "_id" : 15 } );
{ "_id" : { "floatApprox" : 15 }, "count" : 18, "test1" : [
        "ccc",
        [
                "ddd",
                "eee"
        ],
        "fff",
        "ggg",
        [
                "111",
                "222"
        ],
        "444"
], "test2" : [ "ccc" ], "test4" : "testv4", "test5" : "OK" }
> db.test0.update( { "_id" : 15 } , { $pop : { "test1": 1 } } );
> db.test0.find( { "_id" : 15 } );
{ "_id" : { "floatApprox" : 15 }, "count" : 18, "test1" : [ "ccc", [ "ddd", "eee" ], "fff", "ggg", [ "111", "222" ] ], "test2" : [ "ccc" ], "test4" : "testv4",
"test5" : "OK" }

8) $pull

用法：$pull : { field : value } }

从数组field内删除一个等于value值。例：
> db.test0.find( { "_id" : 15 } );
{ "_id" : { "floatApprox" : 15 }, "count" : 18, "test1" : [ "ccc", [ "ddd", "eee" ], "fff", "ggg", [ "111", "222" ] ], "test2" : [ "ccc" ], "test4" : "testv4",
"test5" : "OK" }

> db.test0.update( { "_id" : 15 } , { $pull : { "test1": "ggg" } } );
> db.test0.find( { "_id" : 15 } );
{ "_id" : { "floatApprox" : 15 }, "count" : 18, "test1" : [ "ccc", [ "ddd", "eee" ], "fff", [ "111", "222" ] ], "test2" : [ "ccc" ], "test4" : "testv4", "test5"
 : "OK" }

9) $pullAll

用法：{ $pullAll : { field : value_array } }

同$pull,可以一次删除数组内的多个值。例：
> db.test0.find( { "_id" : 15 } );
{ "_id" : { "floatApprox" : 15 }, "count" : 18, "test1" : [ "ccc", [ "ddd", "eee" ], "fff", [ "111", "222" ] ], "test2" : [ "ccc" ], "test4" : "testv4", "test5"
 : "OK" }

> db.test0.update( { "_id" : 15 } , { $pullAll : { "test1": [ "ccc" , "fff" ] } } );
> db.test0.find( { "_id" : 15 } );
{ "_id" : { "floatApprox" : 15 }, "count" : 18, "test1" : [ [ "ddd", "eee" ], [ "111", "222" ] ], "test2" : [ "ccc" ], "test4" : "testv4", "test5" : "OK" }

10) $ 操作符

$是他自己的意思，代表按条件找出的数组里面某项他自己。呵呵，比较坳口。看一下官方的例子：

> t.find()
{ "_id" : ObjectId("4b97e62bf1d8c7152c9ccb74"), "title" : "ABC",  "comments" : [ { "by" : "joe", "votes" : 3 }, { "by" : "jane", "votes" : 7 } ] }

> t.update( {'comments.by':'joe'}, {$inc:{'comments.$.votes':1}}, false, true )

> t.find()
{ "_id" : ObjectId("4b97e62bf1d8c7152c9ccb74"), "title" : "ABC",  "comments" : [ { "by" : "joe", "votes" : 4 }, { "by" : "jane", "votes" : 7 } ] }

需要注意的是，$只会应用找到的第一条数组项，后面的就不管了。还是看例子：

> t.find();
{ "_id" : ObjectId("4b9e4a1fc583fa1c76198319"), "x" : [ 1, 2, 3, 2 ] }
> t.update({x: 2}, {$inc: {"x.$": 1}}, false, true);
> t.find();

还有注意的是$配合$unset使用的时候，会留下一个null的数组项，不过可以用{$pull:{x:null}}删除全部是null的数组项。例：
> t.insert({x: [1,2,3,4,3,2,3,4]})
> t.find()
{ "_id" : ObjectId("4bde2ad3755d00000000710e"), "x" : [ 1, 2, 3, 4, 3, 2, 3, 4 ] }
> t.update({x:3}, {$unset:{"x.$":1}})
> t.find()
{ "_id" : ObjectId("4bde2ad3755d00000000710e"), "x" : [ 1, 2, null, 4, 3, 2, 3, 4 ] }

{ "_id" : ObjectId("4b9e4a1fc583fa1c76198319"), "x" : [ 1, 3, 3, 2 ] }
```