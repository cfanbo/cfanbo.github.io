---
title: PHP 多台服务器 session 用Memcached存储Session
author: admin
type: post
date: 2013-11-04T02:31:32+00:00
url: /archives/14638
categories:
 - 程序开发
tags:
 - memcached
 - php
 - session

---
**php实现多服务器共享session的方法：**

多服务器共享session的方法：

1.通过NFS文件共享的方式，多台WEB服务器共享保存session文件的磁盘
2.保存在数据库中，这种方式的扩展性很强，可以随意增加WEB而不受影响
3.可以将session数据保存在memcached中，memcached是基于内存存储数据的，性能很高，用户并发量很大的时候尤其合适，参考PHP实现多服务器session共享之memcache共享
4.文件方式保存session时，可以采用php的扩展eaccelerator来存储sesion

php中的Session默认是用文件的方式存储的，如果用多台WEB服务器，Session共享可能就会成为一个大的问题，可以用NFS共享的方式来存储，但是对于并发请求更多的站点来说，用NFS也会出现问题，下面就说说用Memcached来保存Session的问题。

vi memcached_session.php，输入如下的代码

```
$ip = '192.168.1.111';
$port = 11211;
ini_set("session.save_handler", "memcache");
ini_set("session.save_path", "tcp://$ip:$port");

session_start();
$_SESSION['time'] = time();

print 'time:' . $_SESSION['time'];
print "
";
$key = session_id();
print 'session_id:' . $key;
print "
";

$memcache = new Memcache;
$memcache->addServer($ip, $port);
echo "ke: $key value:" . $memcache -> get($key);

```

程序的输出结果如下：
time:1251362605
session_id:7c694d1c2fea5dd3c56f7a19d2f925c9
ke: 7c694d1c2fea5dd3c56f7a19d2f925c9 value:time|i:1251362604;

从上图可以看出，保存在Memcached中的Session数据，key为客户端生成的SessionID，也就是名为PHPSESSID的cookie保存的值，值为Session的值(**$_SESSION[‘time’] = time();**)

上述结果可以说明Session数据已经成功保存到Memcached中了

当然也可以直接在php.ini中修改全局配置，这样，不需要在每个与session有关的程序都需要调用ini_set了

```
session.save_handler = memcache
session.save_path = "tcp://192.168.1.111:11211"

```

使用多个memcached server 时用逗号”,”隔开，并且和 Memcache::addServer() 文档中说明的一样，可以带额外的参数”persistent”、”weight”、”timeout”、”retry_interval” 等等，类似这样的：”tcp://host1:port1?persistent=1&weight=2,tcp://host2:port2″ 。

也可以直接telnet到Memcached中用get session_id来查找Session数据

```
# telnet 192.168.1.111 11211
 Trying 192.168.1.111...
 Connected to 192.168.1.111 (192.168.1.111).
 Escape character is '^]'.
 get 7c694d1c2fea5dd3c56f7a19d2f925c9
 VALUE 7c694d1c2fea5dd3c56f7a19d2f925c9 0 18
 time|i:1251362209;
 END
 set name 0 0 10
 caihuafeng
 STORED
 get name
 VALUE name 0 10
 caihuafeng
 END

```

用 memcache 来存储 session 在读写速度上应该会比文件快很多，而且在多个服务器需要共用 session 时会比较方便，将这些服务器都配置成使用同一组 memcached 服务器就可以，减少了额外的工作量。缺点是 session 数据都保存在内存中，不能持久化存储，如果想持久化存储，可以考虑使用Memcachedb来存储，或用Tokyo Tyrant+Tokyo Cabinet来进行存储。

怎样判断session失效了呢？在php.ini中有个Session.cookie_lifetime的选项，这个代表SessionID在客户端Cookie储存的时间，默认值是“0”，代表浏览器一关闭，SessionID就作废，这样不管保存在Memcached中的Session是否还有效(保存在Memcached中的session会利用Memcached的内部机制进行处理，即使session数据没有失效，而由于客户端的SessionID已经失效，所以这个key基本上不会有机会使用了，利用Memcached的LRU原则，如果Memcached的内存不够用了，新的数据就会取代过期以及最老的未被使用的数据)，因为SessionID已经失效了，所以在客户端会重新生成一个新的SessionID。

保存在Memcached中的数据最长不会超过30天,这个时间是以操作Memcached的时间为基准的，也就是说，只要key还是原来的key,如果你重新对此key进行了相关的操作(如set操作)，且重新设置了有效期，则此时此key对应的数据的有效期会重新计算的,php手册中有说明
Expiration time of the item. If it’s equal to zero, the item will never expire. You can also use Unix timestamp or a number of seconds starting from current time, but in the latter case the number of seconds may not exceed 2592000 (30 days).

Memcached主要的cache机制是LRU（最近最少用）算法+超时失效。当您存数据到memcached中，可以指定该数据在缓存中可以呆多久。如果memcached的内存不够用了，过期的slabs会优先被替换，接着就轮到最老的未被使用的slabs。