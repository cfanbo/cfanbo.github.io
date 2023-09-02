---
title: '[教程]为PHP安装phpRedis扩展模块'
author: admin
type: post
date: 2011-07-12T13:44:06+00:00
url: /archives/10385
IM_contentdowned:
 - 1
categories:
 - 系统架构
 - 服务器
tags:
 - phpize
 - phpredis
 - redis

---
**一.安装phpredis**

> \# wget https://download.github.com/owlient-phpredis-2.1.1-1-g90ecd17.tar.gz
> \# tar -zxvf owlient-phpredis-2.1.1-1-g90ecd17.tar.gz
> \# cd owlient-phpredis-2.1.1-1-g90ecd17
> \# /usr/local/php/bin/phpize
> \# ./configure –with-php-config=/usr/local/php/bin/php-config
> \# make && make install

修改php.ini文件,应用扩展

> \# /usr/local/php/etc/php.ini
> 加入:
> extension=redis.so

重启httpd

> \# service httpd -k restart

我这里使用的是php-fpm模块运行的Nginx

> /usr/local/php/sbin/php-fpm restart

通过phpinfo()函数查看,可以看到redis扩展

[![](http://blog.haohtml.com/wp-content/uploads/2011/07/phpredis.jpg)][1]

如果想用redis来存储session的话,可以这样配置SESSION,不过好像我们用memcached来存储session的比较多一些的,呵呵:

>
> #ini\_set(‘session.save\_handler’, ‘redis’);
> #session\_save\_path(“tcp://host1:6379?weight=1,tcp://host2:6379?weight=2&timeout=2.5,tcp://host3:6379?weight=2”);
>
> ?>

如果要安装最新的php扩展.先安装版本控制.这里用的是git.安装教程请参考: [http://blog.haohtml.com/archives/10093](http://blog.haohtml.com/archives/10093)

> #/usr/local/git/bin/git clone

//下载源码再安装扩展

**二.测试phpredis:**

php测试代码：

>  $redis = new Redis();
> $redis->connect(‘127.0.0.1′,6379);
> $redis->set(‘test’,’hello world!’);
> echo $redis->get(‘test’);
> ?>

输出hello world!

队列测试代码：

入队列操作文件 list_push.php

>  $redis = new Redis();
> $redis->connect(‘127.0.0.1’, 6379);
> while (true) {
> $redis->lPush(‘list1’, ‘A_’.date(‘Y-m-d H:i:s’));
> sleep(rand()%3);
> }
> ?>

执行

\# php list_push.php &

出队列操作 list_pop.php文件

>  $redis = new Redis();
> $redis->pconnect(‘127.0.0.1’, 6379);
> while(true) {
> try {
> var_export( $redis->blPop(‘list1’, 10) );
> } catch(Exception $e) {
> //echo $e;
> }
> }



其他测试代码：

>
> $redis = new Redis();
> $redis->connect(‘127.0.0.1’, 6379);
>
> $redis->set(‘key’, ‘value’);
>
> echo $redis->get(‘key’).”\n”;
>
> $redis->setex(‘key’, 3600, ‘value’); // sets key → value, with 1h TTL.
>
> $redis->set(‘key1’, ‘val1’);
> $redis->set(‘key2’, ‘val2’);
> $redis->set(‘key3’, ‘val3’);
> $redis->set(‘key4’, ‘val4’);
>
> $redis->delete(‘key1’, ‘key2’);
> echo $redis->get(‘key3’).”\n” ;
>
> $redis->delete(array(‘key3’, ‘key4’));
> ?>

[http://code.google.com/p/php-redis/](http://code.google.com/p/php-redis/)

================

**注意事项:**

\# redis目前提供四种数据类型：string,list,set及zset(sorted set)。
\# * string是最简单的类型，你可以理解成与Memcached一模一个的类型，一个key对应一个value，其上支持的操作与Memcached的操 作类似。但它的功能更丰富。
\# * list是一个链表结构，主要功能是push、pop、获取一个范围的所有值等等。操作中key理解为链表的名字。
\# * set是集合，和我们数学中的集合概念相似，对集合的操作有添加删除元素，有对多个集合求交并差等操作。操作中key理解为集合的名字。
\# * zset是set的一个升级版本，他在set的基础上增加了一个顺序属性，这一属性在添加修改元素的时候可以指定，每次指定后，zset会自动重新按新的值调整顺序。可以理解了有两列的mysql表，一列存value，一列存顺序。操作中key理解为zset的名字。

 [1]: http://blog.haohtml.com/wp-content/uploads/2011/07/phpredis.jpg