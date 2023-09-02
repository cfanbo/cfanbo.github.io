---
title: Memcache使用方法（PHP脚本）
author: admin
type: post
date: 2011-03-23T06:02:55+00:00
url: /archives/8095
IM_contentdowned:
 - 1
categories:
 - 程序开发
 - 服务器
tags:
 - memcache

---

Memcache是danga.com的一个项目，最早是为 LiveJournal 服务的，目前全世界不少人使用这个缓存项目来构建自己大负载的网站，来分担数据库的压力。它可以应对任意多个连接，使用非阻塞的网络IO。由于它的工作机 制是在内存中开辟一块空间，然后建立一个HashTable，Memcached自管理这些HashTable。Memcache官方网 站：http://www.danga.com/memcached，更多详细的信息可以来这里了解。


为什么会有Memcache和memcached两种名称？其实Memcache是这个项目的名称，而memcached是它服务器端的主程序文件名，知道我的意思了把~~~~。一个是项目名称，一个是主程序文件名，在网上看到了很多人不明白，于是混用了。


#### Memcache的安装

分为两个过程：memcache服务器端的安装和memcached客户端的安装。


所谓服务器端的安装就是在服务器（一般都是linux系统）上安装Memcache实现数据的存储。


所谓客户端的安装就是指php（或者其他程序，Memcache还有其他不错的api接口提供）去使用服务器端的Memcache提供的函数，需要php添加扩展。

#### PHP的Memcache

`01``<?php`

`02`` ``//连接`

`03`` ``$mem` `= ` `new` `Memcache;`

`04`` ``$mem` `->connect(` `"db.nowamagic.net"` `, 12000);`

`05`

`06`` ``//保存数据`

`07`` ``$mem` `->set(` `'key1'` `, ` `'This is first value'` `, 0, 60);`

`08`` ``$val` `= ` `$mem` `->get(` `'key1'` `);`

`09`` ``echo` `"Get key1 value: "` `. ` `$val` `.` `"<br />"` `;`

`10`

`11`` ``//替换数据`

`12`` ``$mem` `->replace(` `'key1'` `, ` `'This is replace value'` `, 0, 60);`

`13`` ``$val` `= ` `$mem` `->get(` `'key1'` `);`

`14`` ``echo` `"Get key1 value: "` `. ` `$val` `. ` `"<br />"` `;`

`15`

`16`` ``//保存数组`

`17`` ``$arr` `= ` `array` `(` `'aaa'` `, ` `'bbb'` `, ` `'ccc'` `, ` `'ddd'` `);`

`18`` ``$mem` `->set(` `'key2'` `, ` `$arr` `, 0, 60);`

`19`` ``$val2` `= ` `$mem` `->get(` `'key2'` `);`

`20`` ``echo` `"Get key2 value: "` `;`

`21`` ``print_r(` `$val2` `);`

`22`` ``echo` `"<br />"` `;`

`23`

`24`` ``//删除数据`

`25`` ``$mem` `->` `delete` `(` `'key1'` `);`

`26`` ``$val` `= ` `$mem` `->get(` `'key1'` `);`

`27`` ``echo` `"Get key1 value: "` `. ` `$val` `. ` `"<br />"` `;`

`28`

`29`` ``//清除所有数据`

`30`` ``$mem` `->` `flush` `();`

`31`` ``$val2` `= ` `$mem` `->get(` `'key2'` `);`

`32`` ``echo` `"Get key2 value: "` `;`

`33`` ``print_r(` `$val2` `);`

`34`` ``echo` `"<br />"` `;`

`35`

`36`` ``//关闭连接`

`37`` ``$mem` `->close();`

`38``?>`

如果正常的话，浏览器将输出：


`1``Get key1 value: This is first value`

`2``Get key1 value: This is replace value`

`3``Get key2 value: Array ( [0] => aaa [1] => bbb [2] => ccc [3] => ddd )`

`4``Get key1 value:`

`5``Get key2 value:`

#### 程序代码分析

初始化一个Memcache的对象：$mem = new Memcache;


连接到我们的Memcache服务器端，第一个参数是服务器的IP地址，也可以是主机名，第二个参数是Memcache的开放的端口：$mem->connect(“192.168.0.200”, 12000);


保存一个数据到Memcache服务器上，第一个参数是数据的key，用来定位一个数据，第二个参数是需要保存的数据内容，这里是一个字符 串，第三个参数是一个标记，一般设置为0或者MEMCACHE_COMPRESSED就行了，第四个参数是数据的有效期，就是说数据在这个时间内是有效 的，如果过去这个时间，那么会被Memcache服务器端清除掉这个数据，单位是秒，如果设置为0，则是永远有效，我们这里设置了60，就是一分钟有效时 间：$mem->set(‘key1‘, ‘This is first value’, 0, 60);


从Memcache服务器端获取一条数据，它只有一个参数，就是需要获取数据的key，我们这里是上一步设置的key1，现在获取这个数据后输出输出：


`1``$val` `= ` `$mem` `->get(’key1′); `

`2``echo` `"Get key1 value: "` `. ` `$val` `;`

现在是使用replace方法来替换掉上面key1的值，replace方法的参数跟set是一样的，不过第一个参数key1是必须是要替换数据内容的key，最后输出了：


`1``$mem` `->replace(` `'key1'` `, ` `'This is replace value'` `, 0, 60);`

`2``$val` `= ` `$mem` `->get(` `'key1'` `);`

`3``echo` `"Get key1 value: "` `. ` `$val` `;`

同样的，Memcache也是可以保存数组的，下面是在Memcache上面保存了一个数组，然后获取回来并输出：


`1``$arr` `= ` `array` `(` `'aaa'` `, ` `'bbb'` `, ` `'ccc'` `, ` `'ddd'` `);`

`2``$mem` `->set(` `'key2'` `, ` `$arr` `, 0, 60);`

`3``$val2` `= ` `$mem` `->get(` `'key2'` `);`

`4``print_r(` `$val2` `);`

现在删除一个数据，使用delte接口，参数就是一个key，然后就能够把Memcache服务器这个key的数据删除，最后输出的时候没有结果：


`1``$mem` `->` `delete` `(` `'key1'` `);`

`2``$val` `= ` `$mem` `->get(` `'key1'` `);`

`3``echo` `"Get key1 value: "` `. ` `$val` `. ` `"<br />"` `;`

最后我们把所有的保存在Memcache服务器上的数据都清除，会发现数据都没有了，最后输出key2的数据为空，最后关闭连接：


`1``$mem` `->` `flush` `();`

`2``$val2` `= ` `$mem` `->get(` `'key2'` `);`

`3``echo` `"Get key2 value: "` `;`

`4``print_r(` `$val2` `);`

`5``echo` `"<br />"` `;`

#### Memcache的使用

使用Memcache的网站一般流量都是比较大的，为了缓解数据库的压力，让Memcache作为一个缓存区域，把部分信息保存在内存中，在 前端能够迅速的进行存取。那么一般的焦点就是集中在如何分担数据库压力和进行分布式，毕竟单台Memcache的内存容量的有限的。我这里简单提出我的个 人看法，未经实践，权当参考。


**分布式应用**

Memcache本来支持分布式，我们客户端稍加改造，更好的支持。我们的key可以适当进行有规律的封装，比如以user为主的网站来 说，每个用户都有User ID，那么可以按照固定的ID来进行提取和存取，比如1开头的用户保存在第一台Memcache服务器上，以2开头的用户的数据保存在第二胎 Mecache服务器上，存取数据都先按照User ID来进行相应的转换和存取。


但是这个有缺点，就是需要对User ID进行判断，如果业务不一致，或者其他类型的应用，可能不是那么合适，那么可以根据自己的实际业务来进行考虑，或者去想更合适的方法。


**减少数据库压力**

这个算是比较重要的，所有的数据基本上都是保存在数据库当中的，每次频繁的存取数据库，导致数据库性能极具下降，无法同时服务更多的用户， 比如MySQL，特别频繁的锁表，那么让Memcache来分担数据库的压力吧。我们需要一种改动比较小，并且能够不会大规模改变前端的方式来进行改变目 前的架构。


我考虑的一种简单方法：


后端的数据库操作模块，把所有的Select操作提取出来（update/delete/insert不管），然后把对应的SQL进行相应 的hash算法计算得出一个hash数据key（比如MD5或者SHA），然后把这个key去Memcache中查找数据，如果这个数据不存在，说明还没 写入到缓存中，那么从数据库把数据提取出来，一个是数组类格式，然后把数据在set到Memcache中，key就是这个SQL的hash值，然后相应的 设置一个失效时间，比如一个小时，那么一个小时中的数据都是从缓存中提取的，有效减少数据库的压力。缺点是数据不实时，当数据做了修改以后，无法实时到前 端显示，并且还有可能对内存占用比较大，毕竟每次select出来的数据数量可能比较巨大，这个是需要考虑的因素。


#### Memcache的安全

我们上面的Memcache服务器端都是直接通过客户端连接后直接操作，没有任何的验证过程，这样如果服务器是直接暴露在互联网上的话是比 较危险，轻则数据泄露被其他无关人员查看，重则服务器被入侵，因为Mecache是以root权限运行的，况且里面可能存在一些我们未知的bug或者是缓 冲区溢出的情况，这些都是我们未知的，所以危险性是可以预见的。为了安全起见，我做两点建议，能够稍微的防止黑客的入侵或者数据的泄露。


**内网访问**

最好把两台服务器之间的访问是内网形态的，一般是Web服务器跟Memcache服务器之间。普遍的服务器都是有两块网卡，一块指向互联 网，一块指向内网，那么就让Web服务器通过内网的网卡来访问Memcache服务器，我们Memcache的服务器上启动的时候就监听内网的IP地址和 端口，内网间的访问能够有效阻止其他非法的访问。


> # memcached -d -m 1024 -u root -l 192.168.0.200 -p 11211 -c 1024 -P /tmp/memcached.pid

Memcache服务器端设置监听通过内网的192.168.0.200的ip的11211端口，占用1024MB内存，并且允许最大1024个并发连接。


**设置防火墙**

防火墙是简单有效的方式，如果却是两台服务器都是挂在网的，并且需要通过外网IP来访问Memcache的话，那么可以考虑使用防火墙或者 代理程序来过滤非法访问。一般我们在Linux下可以使用iptables或者FreeBSD下的ipfw来指定一些规则防止一些非法的访问，比如我们可 以设置只允许我们的Web服务器来访问我们Memcache服务器，同时阻止其他的访问。


`1``# iptables -F`

`2``# iptables -P INPUT DROP`

`3``# iptables -A INPUT -p tcp -s 192.168.0.2 –dport 11211 -j ACCEPT`

`4``# iptables -A INPUT -p udp -s 192.168.0.2 –dport 11211 -j ACCEPT`

上面的iptables规则就是只允许192.168.0.2这台Web服务器对Memcache服务器的访问，能够有效的阻止一些非法访问，相应的也可以增加一些其他的规则来加强安全性，这个可以根据自己的需要来做。