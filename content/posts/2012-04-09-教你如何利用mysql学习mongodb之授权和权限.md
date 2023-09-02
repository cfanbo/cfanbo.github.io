---
title: 教你如何利用MySQL学习MongoDB之授权和权限
author: admin
type: post
date: 2012-04-09T06:46:23+00:00
url: /archives/12728
IM_data:
 - 'a:1:{s:60:"http://images.51cto.com/files/uploadimg/20110524/0940480.gif";s:67:"http://blog.haohtml.com/wp-content/uploads/2012/04/0331_0940480.gif";}'
IM_contentdowned:
 - 1
categories:
 - nosql
tags:
 - MongoDB

---
在上文中，我们了解了 [教你如何利用MySQL学习MongoDB之SQL语法](http://database.51cto.com/art/201105/263901.htm)，本文中我们继续我们的学习之旅，学习两者的授权和权限。

数据库的安全性是每一个DBA重点关注的部分，在数据库建立之后，数据的安全就显得尤为重要。

对于一个数据库管理员来说，安全性就意味着他必须保证那些具有特殊数据访问权限的用户能够登录到数据库服务器，并且能够访问数据以及对数据库对象实施各种权限范围内的操作;同时，DBA还要防止所有的非授权用户的非法操作。

**1、MySQL授权和权限**

MySQL中有两种级别的权限:管理和用户。所有权限都可分别使用 GRANT 和 REVOKE 语句授予和收回。可以授予用户create、select、update、delete、insert、execute、index 等权限，也可授予alter、drop和shutdown等系统权限。根用户root在默认情况下具有所有权限。

**2、MongoDB授权和权限**

官方文档开启MongoDB 服务时不添加任何参数时，可以对数据库任意操作，而且可以远程访问数据库，所以推荐只是在开发是才这样不设置任何参数。如果启动的时候指定–auth参数,可以从阻止根层面上的访问和连接

**(1)、只允许某ip访问**

mongod –bind_ip 127.0.0.1

**(2)、指定服务端口**

mongod –bind_ip 127.0.0.1 –port27888

**(3)、添加用户认证**

mongod –bind_ip 127.0.0.1 –port27888 –auth

**(4)、添加用户**

在刚安装完毕的时候MongoDB都默认有一个admin数据库，而admin.system.users中将会保存比在其它数据库中设置的用户权限更大的用户信息。

当admin.system.users中一个用户都没有时，即使mongod启动时添加了–auth参数，如果没有在admin数据库中添加用户，此时不进行任何认证还是可以做任何操作，直到在admin.system.users中添加了一个用户。

下面分别创建两个用户, 在foo中创建用户名为user1密码为pwd1的用户，如下：

 1. [root@localhost bin]# ./mongo –port 27888
 2. MongoDB shell version: 1.8.1
 3. connecting to: test
 4. > use foo
 5. switched to db foo
 6. > db.addUser(“user1″,”pwd1”)
 7. {
 8. “user” : “user1”,
 9. “readOnly” : false,
 10. “pwd” : “35263c100eea1512cf3c3ed83789d5e4”
 11. }



在admin中创建用户名为root密码为pwd2的用户，如下：

 1. > use admin
 2. switched to db admin
 3. > db.addUser(“root”, “pwd2”)
 4. {
 5. “_id” : ObjectId(“4f8a87bce495a88dad4613ad”),
 6. “user” : “root”,
 7. “readOnly” : false,
 8. “pwd” : “20919e9a557a9687c8016e314f07df42”
 9. }
 10. > db.auth(“root”, “pwd2”)
 11. 1
 12. >



如果认证成功会显示1, 用以下命令可以查看特定的数据库的用户信息：

 1. > use admin
 2. switched to db admin
 3. > db.system.users.find();
 4. { “_id” : ObjectId(“4f8a87bce495a88dad4613ad”), “user” : “root”, “readOnly” : false, “pwd” : “20919e9a557a9687c8016e314f07df42” }
 5. > use foo
 6. switched to db foo
 7. > db.system.users.find();
 8. { “_id” : ObjectId(“4f92966d77aeb2b2e730c1bb”), “user” : “user1”, “readOnly” : false, “pwd” : “35263c100eea1512cf3c3ed83789d5e4” }
 9. >



下面我们试验一下用户的权限设置是否正确：

 1. [root@localhost bin]# ./mongo –port 27888
 2. MongoDB shell version: 1.8.1
 3. connecting to: 127.0.0.1:27888/test
 4. > use foo
 5. switched to db foo
 6. > db.system.users.find();
 7. error: {
 8. “$err” : “unauthorized db:foo lock type:-1 client:127.0.0.1”,
 9. “code” : 10057
 10. }
 11. > use admin
 12. switched to db admin
 13. > db.system.users.find();
 14. error: {
 15. “$err” : “unauthorized db:admin lock type:-1 client:127.0.0.1”,
 16. “code” : 10057
 17. }
 18. >



通知以上实验结果，说明登录时不指定用户名和口令时会报错，也就是说安全性的部署生效了。下面我再看一下另一个场景：

 1. [root@localhost bin]# ./mongo –port 27888 -uroot -ppwd2
 2. MongoDB shell version: 1.8.1
 3. connecting to: 127.0.0.1:27888/test
 4. Sat Apr 21 19:23:15 uncaught exception: login failed
 5. exception: login failed



奇怪了，我们明明指定了用户名而且口令也没有错呀，这时我们看一下系统日志上是否有一些有价值的信息：

auth: couldn’t find user root, test.system.users



哦，原来是这样，说明连接mongodb时，如果不指定库名，那么会自动连接到test库，但刚才我们新建的用户，都不是在test库上建立的，所以我们需要显示指定需要连接的库名：

 1. [root@localhost bin]# ./mongo –port 27888 admin -uroot -ppwd2
 2. MongoDB shell version: 1.8.1
 3. connecting to: 127.0.0.1:27888/admin
 4. > show collections;
 5. system.indexes
 6. system.users
 7. > use foo
 8. switched to db foo
 9. > show collections
 10. system.indexes
 11. system.users
 12. t1
 13. >



可以看到root这个用户有所有库的操作权限, 那么user1这个用户有什么权限呢?我们一试便知：

 1. [root@localhost bin]# ./mongo –port 27888 foo -uuser1 -ppwd1
 2. MongoDB shell version: 1.8.1
 3. connecting to: 127.0.0.1:27888/foo
 4. > show collections;
 5. system.indexes
 6. system.users
 7. t1
 8. > use test
 9. switched to db test
 10. > show collections
 11. Sat Apr 21 19:28:25 uncaught exception: error: {
 12. “$err” : “unauthorized db:test lock type:-1 client:127.0.0.1”,
 13. “code” : 10057
 14. }
 15. >



通过结果我们看到, 由于user1是在foo库里建立的用户，所以它不具有操作其它数据库，甚至是test库的权限。

转自: