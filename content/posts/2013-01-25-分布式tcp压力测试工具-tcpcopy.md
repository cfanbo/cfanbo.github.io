---
title: 分布式TCP压力测试工具 tcpcopy
author: admin
type: post
date: 2013-01-25T13:41:10+00:00
url: /archives/13618
categories:
 - 服务器
tags:
 - tcpcopy
 - 压力测试

---
tcpcopy是一种应用请求复制（基于tcp的packets）工具，其应用领域较广，我们曾经应用于网易的广告投放系统，urs系统，nginx hmux协议开发等系统，避免了上线带来的很多问题。

**总体说来，tcpcopy主要有如下功能：**

1）分布式压力测试工具，利用在线数据，可以测试系统能够承受的压力大小（远比ab压力测试工具真实地多）,也可以提前发现一些bug
2）对于后端的短连接，请求丢失率非常低(1/10万)，可以应用于热备份
3）普通上线测试，可以发现新系统是否稳定，提前发现上线过程中会出现的诸多问题，让开发者有信心上线
4）对比试验，同样请求，针对不同或不同版本程序，可以做性能对比等试验
5）利用多种手段，构造无限在线压力，满足中小网站压力测试要求
6）实战演习（架构师必备）

tcpcopy可以用于实时和离线回放领域，并且tcpcopy支持mysql协议的复制，开源一年以来，功能上越来越完善。

如果你对上线没有信心，如果你的单元测试不够充分，如果你对新系统不够有把握，如果你对未来的请求压力无法预测，tcpcopy可以帮助你解决上述难题。

=========================

项目地址：



**简介**

tcpcopy是一种请求复制（所有基于tcp的packets）工具，其应用领域较广，我们曾经应用于网易的广告投放系统，urs系统，nginx hmux协议等系统，避免了上线带来的很多问题。

我们即将应用tcpcopy于membase替换现有mecached系统的任务中。由于membase还不够成熟，不适合直接上线，利用tcpcopy程序，可以把访问memcached的系统流量复制一份到membase系统中去。对于membase来说，这份流量就是访问membase的，跟直接上线membase效果一样，就可以做各种试验，查看membase的各种特性。

**tcpcopy六大功能：**

1）分布式压力测试工具，利用在线数据，可以测试系统能够承受的压力大小（远比ab压力测试工具真实地多）,也可以提前发现一些bug
2）如果后端的连接是短连接并且请求体不大，请求丢失率一般都非常低(1/10万)，可以应用于热备份
3）普通上线测试，可以发现新系统是否稳定，提前发现上线过程中会出现的诸多问题，让开发者有信心上线
4）对比试验，同样请求，针对不同或不同版本程序，可以做性能对比等试验
5）利用级联tcpcopy或者利用代理进行多份复制，可以构造无限在线压力，满足中小网站压力测试要求
6）实战演习（架构师必备）

**特点：**

1）实时
2）效果真实
3）低负载，不影响在线
4）操作简单
5）分布式
6）意义非凡

**使用方法：**

TCPCOPY分为TCPCOPY client和TCPCOPY server。
其中TCPCOPY client运行在在线服务器上面，用来捕获在线请求数据包；TCPCOPY server（监听端口为36524）运行在测试机器上面，在测试服务器的响应包丢弃之前截获测试服务器的响应包，并通过TCPCOPY client和TCPCOPY server之间的tcp连接传递响应包的tcp和ip头部信息给TCPCOPY client，以完成TCP交互。

使用方法如下：
TCPCOPY server （root用户执行）
1）启动内核模块ip\_queue (modprobe ip\_queue)
2）设置要截获的端口，并且设置对output截获
iptables -I OUTPUT -p tcp –sport port -j QUEUE
3）./interception
注意（如果已经启动ip_queue和已经设置iptables，只需要运行第3项;测试完以后要记得iptables -F）

TCPCOPY client   （root用户执行）
./tcpcopy 本地ip地址1[：本地ip地址2：…]  本地port  远程ip地址远程port

注意（本地ip地址列表其实就是客户端所认为的服务器ip地址,如果前面有lvs，一般就是lvs的虚拟ip地址）

**测试举例:**

假设13，14是在线应用服务器，148是测试服务器（148配置和13差不多），
本地端口和远程端口都是12321（端口12321是我们的一个应用，类似于memcached应用）。
我们的目的就是为了确认目前在线服务器能否承受目前两倍的压力。

我们利用tcpcopy进行测试：
目标测试服务器(148)
\# modprobe ip_queue (if not run up)
\# iptables -I OUTPUT -p tcp –sport 12321 -j QUEUE (if not set)
\# ./interception
在线服务器(13):
\# ./tcpcopy xx.xx.xx.13 12321 xx.xx.xx.148 12321
在线服务器(14):
\# ./tcpcopy xx.xx.xx.14 12321 xx.xx.xx.148 12321

13 cpu:
11124 adrun 150193m146m744 S 18.67.3495:31.56 asyn_server
11281 root 1506514440m1076 S 12.32.00:47.89 tcpcopy

14 cpu:
16855 adrun 15098.7m55m744 S 21.62.7487:49.51 asyn_server
16429 root 1504115617m1076 S 14.00.90:33.63 tcpcopy

148 cpu :
25609 root 1507689259m764 S 49.62.963:03.14 asyn_server
20184 root 15056244232292 S 17.00.20:52.82 interception

13记录: grep ‘Tue 11:08’ access\_0913\_11.log |wc -l :89316，每秒处理1489次请求

14记录: grep ‘Tue 11:08’ access\_0913\_11.log |wc -l :89309，每秒处理1488次请求

148记录: grep ‘Tue 11:08’ access\_0913\_11.log |wc -l :178175，每秒处理2969次请求

请求丢失率为：(89316+89309-178175)/(89316+89309)=0.25%

从上面可以看出一台在线服务器能够承受目前压力的两倍。
我们来看负载情况：
TCPCOPY client自身负载占到12.3%和14%，TCPCOPY server占到17%，从负载来看，均不高。
内存也占得不多。

**注意事项：**

1）Linux平台，内核2.6+
2）TCPCOPY类似于UDP，所以会丢包，进而丢失请求
3）本系统不支持域名，只支持ip地址
4）LocalRequests，请设置lo MTU不超过1500，并且在配置文件中不要设置127.0.0.1地址，要设置内网或者外网地址
5）TCPCOPY server有可能会成为性能瓶颈
6）丢失请求率跟网络状况有关，最好在内网内复制请求
7）TCPCOPY中的tcpcopy和interception程序运行需要root权限
8）TCPCOPY只与ip、tcp层的数据有关，如果请求验证与tcp层以上的协议有关，则系统不能正常运行。
例如：mysql连接协议，由于权限认证与tcp层上面的mysql协议有关，所以复制过去的请求会被目标测试服务器认为非法请求，这个时候需要针对mysql协议作具体针对性的处理，tcpcopy程序才能正常运行
9）目前追求的是功能，性能优化和代码重构会在稳定以后进行
10）针对长请求（比如上传文件），本系统不是很支持，
因为传递packets到目标测试服务器的时候，没有重传机制的支持（0.4版本会支持重传）
11）如果有问题，请注意error.log文件提示的错误信息(email:163.beijing@gmail.com)
12）源代码已经转到github
13）测试环境最好和在线环境一致，比如连接都保持keepalive
14）客户端ip地址为内网ip地址，一般情况下其应用请求是无法复制到外网测试机器上面去的。
15）tcpcopy client需要连接测试服务器的36524端口，所以要对外开放36524端口
16）为了避免不必要的麻烦，关闭的时候先关闭tcpcopy，然后再关闭interception
17）需要注意TCP segmentation offloading相关问题
如果tcpcopy所抓的数据包大小超过MTU，那么由于raw socket output的原因，需要你
改变在线设置，比如：ethtool -K eth1 tso off ; ethtool -K eth1 gro off
18）如果你想多重复制在线流量，见如下文档(0.3.5+版本）
[http://blog.csdn.net/wangbin579/article/details/7476413](http://blog.csdn.net/wangbin579/article/details/7476413)
19）在测试过程中，如果你还想访问测试服务器的服务(0.3.5+版本），见如下文档
[http://blog.csdn.net/wangbin579/article/details/7476477](http://blog.csdn.net/wangbin579/article/details/7476477)
20）多层架构环境下，测试系统一定要独立，与在线系统没有业务关联，否则会影响在线
21）如果请求丢失率比较高，可以在log.h中设置#define DEBUG_TCPCOPY 1（0.3.2+版本），重新编译，运行，输出若干分钟的log，发送给我。
22）由于更新比较快，高版本针对某些应用，请求丢失率反而有可能会下降，由于需要回归的测试太多，请谅解
23）更多文档和实战，请访问 [http://blog.csdn.net/wangbin579/article/category/926096](http://blog.csdn.net/wangbin579/article/category/926096)
24）个人微博： [http://weibo.com/tcpcopy](http://weibo.com/tcpcopy)