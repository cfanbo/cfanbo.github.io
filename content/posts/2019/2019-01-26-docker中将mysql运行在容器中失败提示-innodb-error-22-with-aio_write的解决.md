---
title: 'docker中将MySQL运行在容器中失败提示“ InnoDB : Error 22 with aio_write”的解决办法'
author: admin
type: post
date: 2019-01-26T03:10:04+00:00
url: /archives/18758
categories:
 - 系统架构
tags:
 - docker

---
今天利用docker容器创建mysql8.0的时候(window系统），指定了本地宿主机器的一个目录为容器mysql的datadir目录，发现创建失败了。

创建命令：

```
$ docker run -d --name mysql81 -v /e/container/mysql/mysql81/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -p 33081:3306 mysql
```

错误提示：

```
$ docker logs mysql81
2019-01-26T03:05:42.567230Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
2019-01-26T03:05:42.567618Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.13) starting as process 1
2019-01-26T03:05:42.572006Z 0 [Warning] [MY-010159] [Server] Setting lower_case_table_names=2 because file system for /var/lib/mysql/ is case insensitive
2019-01-26T03:05:42.832344Z 1 [ERROR] [MY-012592] [InnoDB] Operating system error number 22 in a file operation.
2019-01-26T03:05:42.832473Z 1 [ERROR] [MY-012596] [InnoDB] Error number 22 means 'Invalid argument'
2019-01-26T03:05:42.832556Z 1 [ERROR] [MY-012646] [InnoDB] File ./#innodb_temp/temp_1.ibt: 'aio write' returned OS error 122. Cannot continue operation
2019-01-26T03:05:42.832609Z 1 [ERROR] [MY-012981] [InnoDB] Cannot continue operation.
```

解决办法：

```
docker run -u 1000:50 -d --name mysql81 -v /e/container/mysql/mysql81/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -p 33081:3306 mysql --innodb-use-native-aio=0
```

参数：

-u 表示运行实例的用户，这里的 1000:50 表示的是docker这个用户
–innodb-use-native-aio=0 为MySQL的参数，作用是启用异步操作功能，提高MySQL性能

**注意：**
上面使用了docker运行了mysql实例，则如果进入到容器内部进行一些其它需要高级权限的操作的话，如apt update则会提示权限不足的情况。

参考： [https://github.com/docker-library/mysql/issues/371](https://github.com/docker-library/mysql/issues/371)