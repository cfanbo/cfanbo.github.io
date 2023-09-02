---
title: '使用mysql-proxy实现mysql读写分离[修正于2011-06-23]'
author: admin
type: post
date: 2011-05-18T08:46:36+00:00
url: /archives/9465
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - mysql

---

由于公司数据库负载较大，所以便打算使用读写分离来减轻mysql的负载。目前较为常见的mysql读写分离分为两种：

1、**基于程序代码内部实现**：在代码中根据select、insert进行路由分类；这类方法也是目前生产环境应用最广泛的。优点是性能较好，因为在程序代码中实现，不需要增加额外的设备作为硬件开支。缺点是需要开发人员来实现，运维人员无从下手。

2、**基于中间代理层实现**：我们都知道代理一般是位于客户端和服务器之间，代理服务器接到客户端请求后通过判断然后转发到后端数据库。在这有两个代表性程序

**mysql-proxy****：** [mysql-proxy](http://dev.mysql.com/downloads/mysql-proxy/) 为mysql开源项目，通过其自带的lua脚本进行sql判断，虽然是mysql官方产品，但是mysql官方并不建议将mysql-proxy用到生产环境。

**amoeba****：**由陈思儒开发，作者曾就职于阿里巴巴，现就职于盛大。该程序由java语言进行开发，目前只听说阿里巴巴将其用于生产环境。另外，此项目严重缺少维护和推广（作者有个官方博客，很多用户反馈的问题发现作者不理睬）

经过上述简单的比较，通过程序代码实现mysql读写分离自然是一个不错的选择。但是并不是所有的应用都适合在程序代码中实现读写分离，像大型SNS、B2C这类应用可以在代码中实现，因为这样对程序代码本身改动较小；像一些大型复杂的java应用，这种类型的应用在代码中实现对代码改动就较大了。所以，像这种应用一般就会考虑使用代理层来实现。

下面我们看一下如何搭建mysql-proxy来实现mysql读写分离

**环境拓扑如下：**

[![](http://blog.haohtml.com/wp-content/uploads/2011/05/mysql-proxy-300x138.jpg)][1]

关于mysql、mysql主从的搭建，在此不再演示，如下的操作均在mysql-proxy（192.168.1.200）服务器进行

**一、安装****mysql-proxy**

**1****、安装****lua** (mysql-proxy需要使用lua脚本进行数据转发)

> curl -R -O http://www.lua.org/ftp/lua-5.2.2.tar.gz
> tar zxf lua-5.2.2.tar.gz
> cd lua-5.2.2
> vi Makefile，为了管理方便,修改INSTALL_TOP=
> make linux test

如果安装中遇到

> /usr/bin/ld: cannot find -lreadline
> collect2: ld returned 1 exit status

错误，安装

```
yum install readline-devel
```

即可.

**2****、安装****libevent(****)**

> wget http://monkey.org/~provos/libevent-2.0.12-stable.tar.gz
> tar zxvf libevent-2.0.12-stable.tar.gz
> cd libevent-2.0.12-stable
> ./configure –prefix=/usr/local/libevent
> make && make install

**3****、设置环境变量** （安装mysql-proxy所需变量）

> #vi /etc/profile
>
> export LUA\_CFLAGS=”-I/usr/local/lua/include” LUA\_LIBS=”-L/usr/local/lua/lib -llua -ldl” LDFLAGS=”-L/usr/local/libevent/lib -lm”
> export CPPFLAGS=”-I/usr/local/libevent/include”
> export CFLAGS=”-I/usr/local/libevent/include”
>
> source /etc/profile

**4、安装mysql-proxy()**
[shell]wget http://mysql.cdpa.nsysu.edu.tw/Downloads/MySQL-Proxy/mysql-proxy-0.8.3.tar.gz
tar zxvf mysql-proxy-0.8.3.tar.gz
mv mysql-proxy-0.8.3 /usr/local/mysql-proxy
cd /usr/local/mysql-proxy/[/shell]
**5、启动mysql-proxy**

本次对两台数据库实现了读写分离；mysql-master为可读可写，mysql-slave为只读

> /usr/local/mysql-proxy/bin/mysql-proxy –proxy-backend-addresses=192.168.0.201:3306 –proxy-read-only-backend-addresses=192.168.0.202:3306 –proxy-lua-script=/usr/local/mysql-proxy/share/doc/mysql-proxy/rw-splitting.lua –admin-username=admin –admin-password=admin123 –admin-lua-script=/usr/local/mysql-proxy/share/doc/mysql-proxy/admin-sql.lua &

注：如果正常情况下启动后终端不会有任何提示信息，mysql-proxy启动后会启动两个端口4040和4041.
4040用于SQL转发，4041用于管理mysql-proxy。
如果有多个mysql-slave可以依次在后面添加.

**二、测试**

**1****、连接测试**

因为默认情况下mysql数据库不允许用户在远程连接

> mysql>grant all privileges on \*.\* to ‘root’@’%’ identified by ’123456′;
>
> mysql>flush privileges;

客户端连接

> #mysql -u root -p 123456 -h 192.168.1.200 -P 4040
> #Enter password :

**2****、读写分离测试**

为了测试出mysql读写分离的真实性，在测试之前，需要开启两台mysql的log功能，然后在mysql-slave服务器停止复制.

①  、在两台mysql配置文件my.cnf中的[mysqld]下面加入log=/usr/local/mysql/var/query.log，注意这里的位置.然后重启.

②  、在mysql-slave上执行SQL语句stop slave

③  、在两台mysql上执行#tail -f /usr/local/mysql/var/query.log

④  、在客户端上连接mysql（三个连接以上），然后执行create、select等SQL语句，观察两台mysql的日志有何变化.

注：生产环境中除了进行程序调试外，其它不要开启mysql查询日志，因为查询日志记录了客户端的所有语句，频繁的IO操作将会导致mysql整体性能下降

**总结：**

在上述环境中，mysql-proxy和mysql-master、mysql-slave三台服务器均存在单点故障。如果在可用性要求较高的场合，单点隐患是绝对不允许的。为了避免mysql-proxy单点隐患有两种方法，一种方法是mysql-proxy配合keepalived做双机，另一种方法是将mysql-proxy和应用服务安装到同一台服务器上；为了避免mysql-master单点故障可以使用DRBD+heartbear做双机；避免mysql-slave单点故障增加多台mysql-slave即可，因为mysql-proxy会自动屏蔽后端发生故障的mysql-slave。

在搭建mysql-proxy时遇到不少麻烦，在此再次感谢[**刘兄**][2]和[**师父**][3]的帮助！

摘自:

 [1]: http://blog.haohtml.com/wp-content/uploads/2011/05/mysql-proxy.jpg
 [2]: http://liuyu.blog.51cto.com/
 [3]: http://sery.blog.51cto.com/