---
title: memcached配置
author: admin
type: post
date: 2008-09-24T11:16:01+00:00
excerpt: |
 一、memcached 简介

 在很多场合，我们都会听到 memcached 这个名字，但很多同学只是听过，并没有用过或实际了解过，只知道它是一个很不错的东东。这里简单介绍一下，memcached 是高效、快速的分布式内存对象缓存系统，主要用于加速 WEB 动态应用程序。

 二、memcached 安装

 首先是下载 memcached 了，目前最新版本是 1.1.12，直接从官方网站即可下载到 memcached-1.1.12.tar.gz。除此之外，memcached 用到了 libevent，我下载的是 libevent-1.1a.tar.gz。
url: /archives/364
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - 缓存
 - memcached

---
**一、memcached 简介**

在很多场合，我们都会听到 [memcached][1] 这个名字，但很多同学只是听过，并没有用过或实际了解过，只知道它是一个很不错的东东。这里简单介绍一下，memcached 是高效、快速的分布式内存对象缓存系统，主要用于加速 WEB 动态应用程序。

**二、memcached 安装**

首先是下载 memcached 了，目前最新版本是 1.1.12，直接从官方网站即可下载到 [memcached-1.1.12.tar.gz][2]。除此之外，memcached 用到了 [libevent][3]，我[下载的是 libevent-1.1a.tar.gz][4]。

接下来是分别将 libevent-1.1a.tar.gz 和 memcached-1.1.12.tar.gz 解开包、编译、安装：

```
# tar -xzf libevent-1.1a.tar.gz
# cd libevent-1.1a
# ./configure --prefix=/usr
# make
# make install
# cd ..
# tar -xzf memcached-1.1.12.tar.gz
# cd memcached-1.1.12
# ./configure --prefix=/usr
# make
# make install
```

安装完成之后，memcached 应该在 /usr/bin/memcached。

如果在安装过程中提示：

> checking for libevent directory… configure: error: libevent is required. You can get it from http://www.monkey.org/~provos/libevent/
> If it’s already installed, specify its path using –with-libevent=/dir/

解决办法请参考:．或者直接执行

> yum install libevent-devel

**三、运行 memcached 守护程序**

运行 memcached 守护程序很简单，只需一个命令行即可，不需要修改任何配置文件（也没有配置文件给你修改  ）：

```
/usr/bin/memcached -d -m 128 -l 192.168.1.1 -p 11211 -u root
```

参数解释：

```
-d 以守护程序（daemon）方式运行 memcached；
-m 设置 memcached 可以使用的内存大小，单位为 M；
-l 设置监听的 IP 地址，如果是本机的话，通常可以不设置此参数；
-p 设置监听的端口，默认为 11211，所以也可以不设置此参数；
-u 指定用户，如果当前为 root 的话，需要使用此参数指定用户。
```

当然，还有其它参数可以用，`man memcached` 一下就可以看到了。

**四、memcached 的工作原理**

首先 memcached 是以守护程序方式运行于一个或多个服务器中，随时接受客户端的连接操作，客户端可以由各种语言编写，目前已知的客户端 API 包括 Perl/PHP/Python/Ruby/Java/C#/C 等等。PHP 等客户端在与 memcached 服务建立连接之后，接下来的事情就是存取对象了，每个被存取的对象都有一个唯一的标识符 key，存取操作均通过这个 key 进行，保存到 memcached 中的对象实际上是放置内存中的，并不是保存在 cache 文件中的，这也是为什么 memcached 能够如此高效快速的原因。注意，这些对象并不是持久的，服务停止之后，里边的数据就会丢失。

**三、PHP 如何作为 memcached 客户端**

有两种方法可以使 PHP 作为 memcached 客户端，调用 memcached 的服务进行对象存取操作。

第一种，PHP 有一个叫做 [memcache 的扩展][5]，Linux 下编译时需要带上 –enable-memcache[=DIR] 选项，Window 下则在 php.ini 中去掉 php_memcache.dll 前边的注释符，使其可用。

除此之外，还有一种方法，可以避开扩展、重新编译所带来的麻烦，那就是直接使用 [php-memcached-client][6]。

本文选用第二种方式，虽然效率会比扩展库稍差一些，但问题不大。

**四、PHP memcached 应用示例**

首先 下载 memcached-client.php，在下载了 memcached-client.php 之后，就可以通过这个文件中的类“memcached”对 memcached 服务进行操作了。其实代码调用非常简单，主要会用到的方法有 add()、get()、replace() 和 delete()，方法说明如下：

`add ($key, $val, $exp = 0)`

往 memcached 中写入对象，$key 是对象的唯一标识符，$val 是写入的对象数据，$exp 为过期时间，单位为秒，默认为不限时间；

`get ($key)`

从 memcached 中获取对象数据，通过对象的唯一标识符 $key 获取；

`replace ($key, $value, $exp=0)`

使用 $value 替换 memcached 中标识符为 $key 的对象内容，参数与 add() 方法一样，只有 $key 对象存在的情况下才会起作用；

`delete ($key, $time = 0)`

删除 memcached 中标识符为 $key 的对象，$time 为可选参数，表示删除之前需要等待多长时间。

下面是一段简单的测试代码，代码中对标识符为 ‘mykey’ 的对象数据进行存取操作：

```
<span style="color: #000000;"><span style="color: #0000bb;"><?php </span><span style="color: #ff8000;">//   包含 memcached 类文件 </span><span style="color: #007700;">require_once(</span><span style="color: #dd0000;">‘memcached-client.php’</span><span style="color: #007700;">); </span><span style="color: #ff8000;">//   选项设置 </span><span style="color: #0000bb;">$options </span><span style="color: #007700;">= array(     </span><span style="color: #dd0000;">’servers’ </span><span style="color: #007700;">=> array(</span><span style="color: #dd0000;">‘192.168.1.1:11211′</span><span style="color: #007700;">),/</span><span style="color: #ff8000;">/memcached 服务的地址、端口，可用多个数组元素表示多个 memcached 服务     </span><span style="color: #dd0000;">‘debug’ </span><span style="color: #007700;">=> </span><span style="color: #0000bb;">true</span><span style="color: #007700;">,  </span><span style="color: #ff8000;">//是否打开 debug     </span><span style="color: #dd0000;">‘compress_threshold’ </span><span style="color: #007700;">=> </span><span style="color: #0000bb;">10240</span><span style="color: #007700;">,  </span><span style="color: #ff8000;">//超过多少字节的数据时进行压缩     </span><span style="color: #dd0000;">‘persistant’ </span><span style="color: #007700;">=> </span><span style="color: #0000bb;">false  </span><span style="color: #ff8000;">//是否使用持久连接     </span><span style="color: #007700;">); </span><span style="color: #ff8000;">//   创建 memcached 对象实例 </span><span style="color: #0000bb;">$mc </span><span style="color: #007700;">= new </span><span style="color: #0000bb;">memcached</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$options</span><span style="color: #007700;">); </span><span style="color: #ff8000;">//   设置此脚本使用的唯一标识符 </span><span style="color: #0000bb;">$key </span><span style="color: #007700;">= </span><span style="color: #dd0000;">‘mykey’</span><span style="color: #007700;">; </span><span style="color: #ff8000;">//   往 memcached 中写入对象 </span><span style="color: #0000bb;">$mc</span><span style="color: #007700;">-></span><span style="color: #0000bb;">add</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$key</span><span style="color: #007700;">, </span><span style="color: #dd0000;">’some random strings’</span><span style="color: #007700;">); </span><span style="color: #0000bb;">$val </span><span style="color: #007700;">= </span><span style="color: #0000bb;">$mc</span><span style="color: #007700;">-></span><span style="color: #0000bb;">get</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$key</span><span style="color: #007700;">); echo </span><span style="color: #dd0000;">“n”</span><span style="color: #007700;">.</span><span style="color: #0000bb;">str_pad</span><span style="color: #007700;">(</span><span style="color: #dd0000;">‘$mc->add() ’</span><span style="color: #007700;">, </span><span style="color: #0000bb;">60</span><span style="color: #007700;">, </span><span style="color: #dd0000;">‘_’</span><span style="color: #007700;">).</span><span style="color: #dd0000;">“n”</span><span style="color: #007700;">; </span><span style="color: #0000bb;">var_dump</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$val</span><span style="color: #007700;">); </span><span style="color: #ff8000;">//   替换已写入的对象数据值 </span><span style="color: #0000bb;">$mc</span><span style="color: #007700;">-></span><span style="color: #0000bb;">replace</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$key</span><span style="color: #007700;">, array(</span><span style="color: #dd0000;">’some’</span><span style="color: #007700;">=></span><span style="color: #dd0000;">‘haha’</span><span style="color: #007700;">, </span><span style="color: #dd0000;">‘array’</span><span style="color: #007700;">=></span><span style="color: #dd0000;">‘xxx’</span><span style="color: #007700;">)); </span><span style="color: #0000bb;">$val </span><span style="color: #007700;">= </span><span style="color: #0000bb;">$mc</span><span style="color: #007700;">-></span><span style="color: #0000bb;">get</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$key</span><span style="color: #007700;">); echo </span><span style="color: #dd0000;">“n”</span><span style="color: #007700;">.</span><span style="color: #0000bb;">str_pad</span><span style="color: #007700;">(</span><span style="color: #dd0000;">‘$mc->replace() ’</span><span style="color: #007700;">, </span><span style="color: #0000bb;">60</span><span style="color: #007700;">, </span><span style="color: #dd0000;">‘_’</span><span style="color: #007700;">).</span><span style="color: #dd0000;">“n”</span><span style="color: #007700;">; </span><span style="color: #0000bb;">var_dump</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$val</span><span style="color: #007700;">); </span><span style="color: #ff8000;">//   删除 memcached 中的对象 </span><span style="color: #0000bb;">$mc</span><span style="color: #007700;">-></span><span style="color: #0000bb;">delete</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$key</span><span style="color: #007700;">); </span><span style="color: #0000bb;">$val </span><span style="color: #007700;">= </span><span style="color: #0000bb;">$mc</span><span style="color: #007700;">-></span><span style="color: #0000bb;">get</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$key</span><span style="color: #007700;">); echo </span><span style="color: #dd0000;">“n”</span><span style="color: #007700;">.</span><span style="color: #0000bb;">str_pad</span><span style="color: #007700;">(</span><span style="color: #dd0000;">‘$mc->delete() ’</span><span style="color: #007700;">, </span><span style="color: #0000bb;">60</span><span style="color: #007700;">, </span><span style="color: #dd0000;">‘_’</span><span style="color: #007700;">).</span><span style="color: #dd0000;">“n”</span><span style="color: #007700;">; </span><span style="color: #0000bb;">var_dump</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$val</span><span style="color: #007700;">); </span><span style="color: #0000bb;">?> </span></span>
```

是不是很简单，在实际应用中，通常会把数据库查询的结果集保存到 memcached 中，下次访问时直接从 memcached 中获取，而不再做数据库查询操作，这样可以在很大程度上减轻数据库的负担。通常会将 SQL 语句 md5() 之后的值作为唯一标识符 key。下边是一个利用 memcached 来缓存数据库查询结果集的示例（此代码片段紧接上边的示例代码）：

```
<span style="color: #000000;"><span style="color: #0000bb;"><?php $sql </span><span style="color: #007700;">= </span><span style="color: #dd0000;">‘SELECT * FROM users’</span><span style="color: #007700;">; </span><span style="color: #0000bb;"> $key </span><span style="color: #007700;">= </span><span style="color: #0000bb;">md5</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$sql</span><span style="color: #007700;">);   </span><span style="color: #ff8000;"> //memcached 对象标识符 </span><span style="color: #007700;">if ( !(</span><span style="color: #0000bb;">$datas </span><span style="color: #007700;">= </span><span style="color: #0000bb;">$mc</span><span style="color: #007700;">-></span><span style="color: #0000bb;">get</span><span style="color: #007700;">(</span><span style="color: #0000bb;">$key</span><span style="color: #007700;">)) ) {     </span><span style="color: #ff8000;">//   在 memcached 中未获取到缓存数据，则使用数据库查询获取记录集。 </span><span style="color: #007700;"><span class="code" style="padding-bottom: 20px; width: 500px;"><code><span style="color: #ff8000;">     </span></span>echo </span>“n”.str_pad(‘Read datas from MySQL.’, 60, ‘_’).“n”;     $conn = mysql_connect(‘localhost’, ‘test’, ‘test’);      mysql_select_db(‘test’);     $result = mysql_query($sql);      while ($row = mysql_fetch_object($result))         $datas[] = $row;</span>     //   将数据库中获取到的结果集数据保存到 memcached 中，以供下次访问时使用。     $mc->add($key, $datas); } else {      echo “n”.str_pad(‘Read datas from memcached.’, 60, ‘_’).“n”; } var_dump($datas); ?>  </code>
```

可以看出，使用 memcached 之后，可以减少数据库连接、查询操作，数据库负载下来了，脚本的运行速度也提高了。

之前我曾经写过一篇名为[《PHP 实现多服务器共享 SESSION 数据》][7]文章，文中的 SESSION 是使用数据库保存的，在并发访问量大的时候，服务器的负载会很大，经常会超出 MySQL 最大连接数，利用 memcached，我们可以很好地解决这个问题，工作原理如下：

 * 用户访问网页时，查看 memcached 中是否有当前用户的 SESSION 数据，使用 session_id() 作为唯一标识符；如果数据存在，则直接返回，如果不存在，再进行数据库连接，获取 SESSION 数据，并将此数据保存到 memcached 中，供下次使用；
 * 当前的 PHP 运行结束（或使用了 [session_write_close()][8]）时，会调用 My_Sess::write() 方法，将数据写入数据库，这样的话，每次仍然会有数据库操作，对于这个方法，也需要进行优化。使用一个全局变量，记录用户进入页面时的 SESSION 数据，然后在 write() 方法内比较此数据与想要写入的 SESSION 数据是否相同，不同才进行数据库连接、写入数据库，同时将 memcached 中对应的对象删除，如果相同的话，则表示 SESSION 数据未改变，那么就可以不做任何操作，直接返回了；
 * 那么用户 SESSION 过期时间怎么解决呢？记得 memcached 的 add() 方法有个过期时间参数 $exp 吗？把这个参数值设置成小于 SESSION 最大存活时间即可。另外别忘了给那些一直在线的用户延续 SESSION 时长，这个可以在 write() 方法中解决，通过判断时间，符合条件则更新数据库数据。

 [1]: http://www.danga.com/memcached/
 [2]: http://www.danga.com/memcached/dist/memcached-1.1.12.tar.gz
 [3]: http://monkey.org/~provos/libevent/
 [4]: http://monkey.org/~provos/libevent-1.1a.tar.gz
 [5]: http://www.php.net/manual/en/ref.memcache.php
 [6]: http://wikipedia.sourceforge.net/doc/memcached-client/_includes_memcached-client_php.html
 [7]: http://www.infor96.com/~nio/sharing-php-session-data-between-servers/
 [8]: http://php.liukang.com/manual/en/function.session-write-close.php