---
title: nginx 的限制连接模块limit_zone与limit_req_zone
author: admin
type: post
date: 2012-03-25T08:19:16+00:00
url: /archives/12640
IM_data:
 - 'a:7:{s:53:"http://img1.51cto.com/attachment/201108/140650433.jpg";s:53:"http://img1.51cto.com/attachment/201108/140650433.jpg";s:53:"http://img1.51cto.com/attachment/201108/140727549.jpg";s:53:"http://img1.51cto.com/attachment/201108/140727549.jpg";s:53:"http://img1.51cto.com/attachment/201108/142708172.jpg";s:53:"http://img1.51cto.com/attachment/201108/142708172.jpg";s:53:"http://img1.51cto.com/attachment/201108/144303203.jpg";s:53:"http://img1.51cto.com/attachment/201108/144303203.jpg";s:53:"http://img1.51cto.com/attachment/201108/141228407.jpg";s:53:"http://img1.51cto.com/attachment/201108/141228407.jpg";s:53:"http://img1.51cto.com/attachment/201108/141251535.jpg";s:53:"http://img1.51cto.com/attachment/201108/141251535.jpg";s:53:"http://img1.51cto.com/attachment/201108/141406358.jpg";s:53:"http://img1.51cto.com/attachment/201108/141406358.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nginx

---
nginx 上有两个限制连接的模块一个是 limit\_zone 另一个是 limie\_req_zone,两个都可以限制连接，但具体有什么不同呢？
下面是 nginx 官网上给的解释

> limit\_req\_zone
> Limit frequency of connections from a client.
> This module allows you to limit the number of requests for a given session, or as a special case, with one address.
> Restriction done using leaky bucket.
>
> limit_zone
> Limit simultaneous connections from a client.
> This module makes it possible to limit the number of simultaneous connections for the assigned session or as a special case, from one address.

按照字面的理解，lit\_req\_zone的功能是通过 令牌桶原理来限制 用户的连接频率，(这个模块允许你去限制单个地址 指定会话或特殊需要 的请求数 )
而 limit_zone 功能是限制一个客户端的并发连接数。(这个模块可以限制单个地址 的指定会话 或者特殊情况的并发连接数)


一个是限制并发连接一个是限制连接频率，表面上似乎看不出来有什么区别，那就看看实际的效果吧~~~
在我的测试机上面加上这两个参数下面是我的部分配置文件

> http{
> limit\_zone one  $binary\_remote_addr  10m;
> #limit\_req\_zone  $binary\_remote\_addr  zone=req_one:10m rate=1r/s;
> server
> {
> ……
> limit_conn   one  1;
> #limit\_req   zone=req\_one  burst=120;
> ……
> }
> }

解释一下 limit\_zone one  $binary\_remote_addr  10m;
这里的 one 是声明一个 limit\_zone 的名字，$binary\_remote\_addr是替代 $remore\_addr 的变量，10m 是会话状态储存的空间
limit_conn one 1 ,限制客户端并发连接数量为1
先测试 limit_zone 这个模块
我找一台机器 用ab 来测试一下 命令格式为

> ab -c 100 -t 10 http://192.168.6.26/test.php

test.php 内容是
看看日志里的访问

[![](http://img1.51cto.com/attachment/201108/140650433.jpg)](http://img1.51cto.com/attachment/201108/140650433.jpg)

看来也不一定能限制的住1秒钟1个并发连接，（有网友跟我说这是因为测试的文件本身太小了才会这样，有时间一定测试一下），从日志里面可以看得出来 除了几个200以外其他的基本都是503，多数并发访问都被503了。

我又用ab多运行了一会儿，发现另一种情况

[![](http://img1.51cto.com/attachment/201108/140727549.jpg)](http://img1.51cto.com/attachment/201108/140727549.jpg)

似乎随着数量的增多效果也会发生一些变化，并不是完全达到模块说明中的效果
看看当前的tcp连接数

> \# netstat -n | awk ‘/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}’
> TIME_WAIT 29
> FIN_WAIT1 152
> FIN_WAIT2 2
> ESTABLISHED 26
> SYN_RECV 16

这次测试下 limit\_req\_zone,配置文件稍微改动一下

> http{
> #limit\_zone one  $binary\_remote_addr  10m;
> limit\_req\_zone  $binary\_remote\_addr  zone=req_one:10m rate=1r/s;
> server
> {
> ……
> #limit_conn   one  1;
> limit\_req   zone=req\_one  burst=120;
> ……
> }
> }

restart 一下 nginx
简单说明一下， rate=1r/s 的意思是每个地址每秒只能请求一次，也就是说根据令牌桶(经过网友冰冰的指正应该是漏桶原理)原理 burst=120 一共有120块令牌，并且每秒钟只新增1块令牌，
120块令牌发完后 多出来的那些请求就会返回503
测试一下

> ab -c 100 -t 10 http://192.168.6.26/test.php

看看这时候的访问日志 [![](http://img1.51cto.com/attachment/201108/142708172.jpg)](http://img1.51cto.com/attachment/201108/142708172.jpg)

确实是每秒请求一次，那多测试一会儿呢？把时间从10秒增加到30秒

[![](http://img1.51cto.com/attachment/201108/144303203.jpg)](http://img1.51cto.com/attachment/201108/144303203.jpg)

这个时候应该是120 已经不够用了，出现很多503，还有两种情况会出现,请看图

[![](http://img1.51cto.com/attachment/201108/141228407.jpg)](http://img1.51cto.com/attachment/201108/141228407.jpg)

这种情况很像是 在队列里的一些请求得不到响应而超时了，但我不确定是不是这种情况。

[![](http://img1.51cto.com/attachment/201108/141251535.jpg)](http://img1.51cto.com/attachment/201108/141251535.jpg)

客户端自己等不及断开了，返回499
看看当前的tcp连接数

> netstat -n | awk ‘/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}’
> TIME_WAIT 51
> FIN_WAIT1 5
> ESTABLISHED 155
> SYN_RECV 12

虽然这样会让nginx 一秒钟只处理一个请求，但是仍然会有很多还在队列里面等待处理，这样也会占用很多tcp连接，从上面那条命令的结果中就能看得出来。
如果这样呢

> limit\_req   zone=req\_one  burst=120 nodelay;

加上 nodelay之后超过 burst大小的请求就会直接 返回503，如图

[![](http://img1.51cto.com/attachment/201108/141406358.jpg)](http://img1.51cto.com/attachment/201108/141406358.jpg)

也是每秒处理1个请求，但多出来的请求没有象刚才那样等待处理，而是直接返回503。

当前的tcp连接

> \# netstat -n | awk ‘/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}’
> TIME_WAIT 30
> FIN_WAIT1 15
> SYN_SENT 7
> FIN_WAIT2 1
> ESTABLISHED 40
> SYN_RECV 37

已连接的数量比上面的少了一些
通过这次测试我发现 这两种模块都不能做到绝对的限制，但的确已经起到了很大的减少并发和限制连接的作用，在生产环境中具体用哪种或者需要两种在一起使用就要看各自的需求了。
测试就到这里，如果文章里有不对的地方请大家及时指正，谢谢

本文出自 “[story的天空][1]” 博客，请务必保留此出处

 [1]: http://storysky.blog.51cto.com/