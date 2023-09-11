---
title: MySQL中select中的for update 的用法
author: admin
type: post
date: 2018-08-04T07:04:40+00:00
url: /archives/18027
categories:
 - MySQL
tags:
 - mysql

---

```
注意: FOR UPDATE 只能用在事务区块(BEGIN/COMMIT)中才有效。
```

有时候我们会看到一些select语句后面紧跟一句for update，表示手动加锁的意思，这里我们就介绍一下对for update的理解。相对另一种手动加锁方法lock in share mode 的区别见： [https://blog.csdn.net/liangzhonglin/article/details/65438777](https://blog.csdn.net/liangzhonglin/article/details/65438777)。

for update：IX锁(意向排它锁)，即在符合条件的rows上都加了排它锁，其他session也就无法在这些记录上添加任何的S锁或X锁。如果不存在一致性非锁定读的话，那么其他session是无法读取和修改这些记录的，但是innodb有非锁定读(快照读并不需要加锁)，for update之后并不会阻塞其他session的快照读取操作，除了select …lock in share mode和select … for update这种显示加锁的查询操作。

lock in share mode：是IS锁(意向共享锁)，即在符合条件的rows上都加了共享锁，这样的话，其他session可以读取这些记录，也可以继续添加IS锁，但是无法修改这些记录直到你这个加锁的session执行完成(否则直接锁等待超时)。

for update是一个意向排它锁，也就是说对于select … for update 这条语句所在的事务中可以进行任何操作(锁定的是记录，这里指主键id=1的这条记录)，但其它事务中只能读取(对这条id=1的记录），不能进行update更新操作。

一般用在并发场景下，如双11的时候商品数量的更新，如果不添加for update的话，则会出现商品数量被多减的bug。

为了更加方便理解，我们举例说明(事务隔离级别为 RR，这里表tb的id为主键)

事务A:

```
start transaction;
select * from tb where id=1 for update;
update tb set product_num=product_num-1 where id=1;
```

此时另一个事务B执行同样的程序语句:

```
start transaction;
// 下面此时会被阻塞，直到事务A提交或者回滚
select * from tb where id=1 for update;
update tb set product_num=product_num-1 where id=1;

```

对于其它主键值非1的不存在这种情况，只要两个事务操作的不是同一条记录就可以执行成功。

推荐阅读： [https://blog.csdn.net/u011957758/article/details/75212222](https://blog.csdn.net/u011957758/article/details/75212222)