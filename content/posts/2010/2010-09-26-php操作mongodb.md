---
title: PHP操作MongoDB
author: admin
type: post
date: 2010-09-26T02:06:55+00:00
url: /archives/5815
IM_contentdowned:
 - 1
categories:
 - 程序开发
 - 数据库
tags:
 - MongoDB

---
**一、MongoDB简介**

MongoDB (名称来自”humongous”) 是一个可扩展的、高性能、开源、模式自由、面向文档的数据库，集文档数据库、键值对存储和关系型数据库的优点于一身。官方站点： [http://www.mongodb.org/](http://www.mongodb.org/)，MongoDB特点：

 * 面向文档存储(类JSON数据模式简单而强大)
 * 动态查询
 * 全索引支持,扩展到内部对象和内嵌数组
 * 查询记录分析
 * 快速,就地更新
 * 高效存储二进制大对象 (比如照片和视频)
 * 复制和故障切换支持
 * Auto-Sharding自动分片支持云级扩展性
 * MapReduce 支持复杂聚合
 * 商业支持,培训和咨询

**二、安装MongoDB**

安装MongoDB非常的简单，仅需下载压缩包解压运行命令即可，下载地址： [http://www.mongodb.org/downloads](http://www.mongodb.org/downloads)，本文为windows平台，MongoDB运行命令：>bin/mongod。提示：首先要创建存储数据的文件夹，MongoDB 默认存储数据目录为 /data/db/ (或者 c:datadb)，当然你也可以修改成不同目录，只需要指定 –dbpath 参数，eg：

>bin/mongod –dbpath=d:mgdatadb

**参考:** Windows下MongoDB安装教程: [http://blog.haohtml.com/archives/5818](http://blog.haohtml.com/archives/5818)

**三、安装MongoDB PHP扩展**



根据自己的PHP版本下载PHP扩展： [http://github.com/mongodb/mongo-php-driver/downloads](http://github.com/mongodb/mongo-php-driver/downloads)，提示：1、VC6适合Apache、VC9适合IIS；

2、Thread safe适合PHP以模块运行方式、Non-thread safe适合CGI运行方式。

修改php.ini，加入：extension=php_mongo.dll，重启Web服务器。

**四、PHP示例**

1、连接Mongo服务器

```
<?php
//连接localhost:27017
$conn = new Mongo();
//连接远程主机默认端口
$conn = new Mongo('test.com');
//连接远程主机22011端口
$conn = new Mongo('test.com:22011');
//MongoDB有用户名密码
$conn = new Mongo("mongodb://${username}:${password}@localhost")
//MongoDB有用户名密码并指定数据库blog
$conn = new Mongo("mongodb://${username}:${password}@localhost/blog");
//多个服务器
$conn = new Mongo("mongodb://localhost:27017,localhost:27018");
?>

```

2、指定数据库和数据集名（表名）

```
<?php
//选择数据库blog
$db = $conn->blog;
//制定结果集（表名：users）
$collection = $db->users;
?>

```

3、CRUD

```
<?php
//新增
$user = array('name' => 'caleng', 'email' => 'admin@admin.com');
$collection->insert($user);
//修改
$newdata = array('$set' => array("email" => "test@test.com"));
$collection->update(array("name" => "caleng"), $newdata);
//删除
$collection->remove(array('name'=>'caleng'), array("justOne" => true));
//查找
$cursor = $collection->find();
var_dump($cursor);
//查找一条
$user = $collection->findOne(array('name' => 'caleng'), array('email'));
var_dump($user);
?>

```

4、关闭连接

```
<?php
$conn->close();
?>

```

**更多教程：** [http://www.php.net/manual/en/book.mongo.php](http://www.php.net/manual/en/book.mongo.php)