---
title: Linux下安装MySQL多实例
author: admin
type: post
date: 2016-12-03T15:12:50+00:00
url: /archives/17300
categories:
 - MySQL

---
**环境说明：**
Centos 6.6 64位
mysql 使用最新版本5.7.16版本

这里安装两个MySQL实例，分别使用3306/3307端口号

**目录结构：**
/data/mysql/mysql3306
/data/mysql/mysql3306/data
/data/mysql/mysql3307/log
/data/mysql/mysql3306/tmp

执行命令：

```
mkdir -p /data/mysql/mysql3306/{data,tmp,log}
mkdir -p /data/mysql/mysql3307/{data,tmp,log}

```

为了方便我们先配置mysql3306实例，配置成功后，再复制一份到3307即可。

```
tar zxvf mysql-5.7.16-linux-glibc2.5-x86_64.tar.gz
cp -rf mysql-5.7.16-linux-glibc2.5-x86_64/* /data/mysql/mysql3306/

```

权限修改

```
chown -R mysql:mysql /data/mysql/mysql3306

```

配置my.cnf

```
cd /data/mysql/mysql3306
cp support-files/my-default.cnf ./my.cnf

```

编辑/data/mysql/mysql3306/my.cnf 内容如下：

```
[client]
port=3306

[mysqld]
basedir=/data/mysql/mysql3306
datadir=/data/mysql/mysql3306/data
socket=/data/mysql/mysql3306/tmp/mysql.sock
port=3306

user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

[mysqld_safe]
log-error=/data/mysql/mysql3306/log/mysqld.log
pid-file=/data/mysql/mysql3306/mysqld.pid

```

初始化表，并生成一个随机root密码

```
bin/mysqld --initialize --basedir=/data/mysql/mysql3306 --datadir=/data/mysql/mysql3306/data

2016-12-03T13:33:57.097171Z 1 [Note] A temporary password is generated for root@localhost: y+_;!l#uh3TK

```

启用mysql 实例

```
bin/mysqld_safe --defaults-file=/data/mysql/mysql3306/my.cnf --user=mysql &

```

确认是否安装成功

```
ps aux | grep mysql

```

如果一切顺利的话，会看到mysql进程和启动配置项。

如果想停止服务的话，可以执行

```
bin/mysqladmin -uroot -p shutdown -S /data/mysql/mysql3306/tmp/mysql.sock
```

到此这个实例安装成功了。

下面我们来测试一下MySQL.如果直接在本机使用客户端的话，会提示找不到 /tmp/mysql.sock 文件，需要加上 -S 参数指定sock文件路径才可以。如连接mysql3306实例：

```
mysql -u root -S /data/mysql/mysql3306/tmp/msql.sock -p
```

这样就可以连接到3306端口。

我们用同样的方法安装mysql3307实例。如果想直接复制mysql3306目录的话（cp -rf mysql3306 mysql3307），记得要先停止mysql3306实例服务。复制完要将data目录里清空，不然没有办法进行初始化表操作。另外还需要注意的有两点：
一个是mysql目录的权限要执行chown -R mysql:mysql /data/mysql/mysql3307

另一个就是配置文件my.cnf ，记得修改端口号为3307 和 路径为mysql3307目录

* * *

我们可以从另一个机器连接到数据库，在连接的时候指 -h 参数即可（默认root不允许远程登录，记得授权）。如：

```
mysql -u root -h 192.168.0.45 -P 3307 -p
```

输入密码即可成功,如果端口号为3306则可以省略-P参数。