---
title: '[教程]memcached for win32的安装'
author: admin
type: post
date: 2008-09-24T15:56:06+00:00
excerpt: |
 memcached是由livejournal团队(danga.com)制作的开源缓存软件，是缓存机制的一种实现，用它之所以高效，是因为它是利用了内存，使用好了能够大大加快页面或者是其它程序的执行速度。要注意的是一旦服务器停止，内存中的缓存数据会被清空。

 win32下，需要启动memcached服务，首先下载相关的memcached文件（用于启动服务的windows.rar在附件中），解压后可以自己选择，这里我选择的是2.1版本的，将其中的memcached.exe和memcached.ini（里面也就这俩文件）拷贝到某路径下（如：E:\java\memcached2.1），然后通过cmd命令窗口，先转入到该路径，然后按如下步骤输入：
url: /archives/408
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - 缓存
 - memcached
 - php_memcache.dll

---

memcached是由livejournal团队(danga.com)制作的开源缓存软件，是缓存机制的一种实现，用它之所以高效，是因为它是利用了内存，使用好了能够大大加快页面或者是其它程序的执行速度。要注意的是一旦服务器停止，内存中的缓存数据会被清空。


win32下，需要启动memcached服务，首先下载相关的memcached文件（用于启动服务的windows.rar在附件中），解压后可以自己选择，这里我选择的是2.1版本的，将其中的memcached.exe和memcached.ini（里面也就这俩文件）拷贝到某路径下（如：E:javamemcached2.1），然后通过cmd命令窗口，先转入到该路径，然后按如下步骤输入：

1、memcached.exe -d install

2、memcached.exe -d start

这里第一步是用于安装服务，第二步是用于启动服务，有些默认参数的值是通过memcached.ini里的相关元素的设置值而定的。

如果要停止服务和卸载服务可以用入下命令：


3、memcached.exe -d stop 或 memcached.exe -d shutdown

4、memcached.exe -d uninstall

**命令参数解释：**

 -d 以守护程序（daemon）方式运行 memcached


 -m 设置 memcached 可以使用的内存大小，单位为 M


 -l 设置监听的 IP 地址，如果是本机的话，通常可以不设置此参数


 -p 设置监听的端口，默认为 11211，所以也可以不设置此参数


 -u 指定用户，如果当前为 root 的话，需要使用此参数指定用户

 -p 监听的端口

 -l 连接的IP地址, 默认是本机

 -d start 启动memcached服务

 -d restart 重起memcached服务

 -d stop|shutdown 关闭正在运行的memcached服务

 -d install 安装memcached服务

 -d uninstall 卸载memcached服务

 -u 以的身份运行 (仅在以root运行的时候有效)

 -m 最大内存使用，单位MB。默认64MB

 -M 内存耗尽时返回错误，而不是删除项

 -c 最大同时连接数，默认是1024

 -f 块大小增长因子，默认是1.25(增长因子: [http://zhengjunwei2007-163-com.iteye.com/blog/963375](http://zhengjunwei2007-163-com.iteye.com/blog/963375))

 -n 最小分配空间，key+value+flags默认是48

 -h 显示帮助


memcached.ini文件指定的各参数及值可以根据需要做更改，默认的如下：

bind_addr=127.0.0.1

listener_port=11212

memory=16

max_conns=1024

evict_to_free = 0

说明：


bind_addr是指绑定的IP

listener_port是指监听的端口


memory内存大小


max_conns最大连接数


evict_to_free


在通过命令 memcached.exe -d start 将服务启动后，可以通过一个demo来做测试，这个demo的下载地址如下：

[http://www.whalin.com/memcached/#download](http://www.whalin.com/memcached/#download)

运行类com.danga.MemCached.test.TestMemcached前将服务IP和监听端口改为启动服务时所用的IP和监听端口，

如：String[] servers = { “127.0.0.1:11212”};

或者可以用如下的Test类做测试


java 代码


01. package com.danga.MemCached.test;
03. import java.util.Date;
05. import com.danga.MemCached.MemCachedClient;
06. import com.danga.MemCached.SockIOPool;
09. publicclass Test {
10. protectedstatic MemCachedClient mcc = new MemCachedClient();
12. static {
13.         String[] servers ={“127.0.0.1:11212”};
15.         Integer[] weights = { 3 };
17. //创建一个实例对象SockIOPool
18.         SockIOPool pool = SockIOPool.getInstance();
20. // set the servers and the weights
21. //设置Memcached Server
22.         pool.setServers( servers );
23.         pool.setWeights( weights );
25. // set some basic pool settings
26. // 5 initial, 5 min, and 250 max conns
27. // and set the max idle time for a conn
28. // to 6 hours
29.         pool.setInitConn( 5 );
30.         pool.setMinConn( 5 );
31.         pool.setMaxConn( 250 );
32.         pool.setMaxIdle( 1000 * 60 * 60 * 6 );
34. // set the sleep for the maint thread
35. // it will wake up every x seconds and
36. // maintain the pool size
37.         pool.setMaintSleep( 30 );
39. //        Tcp的规则就是在发送一个包之前，本地机器会等待远程主机
40. //        对上一次发送的包的确认信息到来；这个方法就可以关闭套接字的缓存，
41. //        以至这个包准备好了就发；
42.         pool.setNagle( false );
43. //连接建立后对超时的控制
44.         pool.setSocketTO( 3000 );
45. //连接建立时对超时的控制
46.         pool.setSocketConnectTO(  );
48. // initialize the connection pool
49. //初始化一些值并与MemcachedServer段建立连接
50.         pool.initialize();
53. // lets set some compression on for the client
54. // compress anything larger than 64k
55.         mcc.setCompressEnable( true );
56.         mcc.setCompressThreshold( 64 * 1024 );
57.     }
59. publicstaticvoid bulidCache(){
60. //set(key,value,Date) ,Date是一个过期时间，如果想让这个过期时间生效的话，这里传递的new Date(long date) 中参数date，需要是个大于或等于1000的值。
61. //因为java client的实现源码里是这样实现的 expiry.getTime() / 1000 ，也就是说，如果 小于1000的值，除以1000以后都是0，即永不过期
62.         mcc.set( “test”, “This is a test String” ,new Date(10000));   //十秒后过期
64.     }
66. publicstaticvoid output() {
67. //从cache里取值
68.         String value = (String) mcc.get( “test” );
69.         System.out.println(value);
70.     }
72. publicstaticvoid main(String[] args){
73.         bulidCache();
74.         output();
75.     }
77. }

**应用memcached的一个心得：**

1、客户端在与 memcached 服务建立连接之后，进行存取对象的操作,每个被存取的对象都有一个唯一的标识符 key，存取操作均通过这个 key 进行，保存到 memcached 中的对象实际上是放置内存中的，并不是保存在 cache 文件中的，这也是为什么 memcached 能够如此高效快速的原因。注意，这些对象并不是持久的，服务停止之后，里边的数据就会丢失。


2、当存入cached的数据超过了cached的容量后会将最长时间没调用的对象挤出，这正好应征了cached的特征。


3、利用memcached常用的做法：在每取得一次cached对象后，重新设置这个对象的cache时间，这样能够使得经常被调用的对象可以长期滞留在缓存中，使得效率增倍。


[windows.rar (](/wp-content/uploads/2011/10/memcached_for_windows.rar) 2.5 MB)


**相关教程:**

Linux下Memcached的安装及配置: [http://blog.haohtml.com/archives/364](http://blog.haohtml.com/archives/364)

基于 PHP5 & JQuery 的 Memcached 管理监控工具: [http://www.junopen.com/memadmin/](http://www.junopen.com/memadmin/)