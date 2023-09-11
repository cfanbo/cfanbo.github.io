---
title: 使用docker-compose 快速创建一个mysql 数据库容器
author: admin
type: post
date: 2018-04-20T04:57:49+00:00
url: /archives/17775
categories:
 - 系统架构
tags:
 - docker

---
**//创建一个独立的容器目录**

```
mkdir docker-db
cd docker-db

```

**前提、创建 docker Compose 配置文件**
#vi docker-compose.yml 文件,内容如下

```
version: '3.6'
services:

    db:
        image: hub.c.163.com/library/mysql:5.7
        restart: always
        environment:
            MYSQL_ROOT_PASSWORD: 123456
            MYSQL_DATABASE: wordpress
            MYSQL_USER: root
            MYSQL_PASSWORD: 123456
            MYSQL_ALLOW_EMPTY_PASSWORD: "yes"
        ports:
            - "33061:3306"

```

image也可以直接写mysql:5.7 或者mysql:latest，不指定获取地址。
上面的 ports 这一块，是指宿主机端口号:容器端口号。在使用的时候，直接访问本机的 33061 端口即可。端口号前也可以指定一个固定的IP 地址。
Compose file version 3 reference： [https://docs.docker.com/compose/compose-file/](https://docs.docker.com/compose/compose-file/)

**一、创建并启动容器**

```
$docker-composer up
```

在 docker-composr.yml 所在的目录里，执行上面的命令，此时会自动从远程服务器拉取容器所需的信息。这时窗口一直处于运行状态，我们通过添加
**-d**
参数，来实现后台服务运行。

**二、停止关闭容器**

```
$ docker-compose stop

```

关闭后，容器文件仍然在磁盘上存在，重新执行 docker-compose start 即可启动。

**三、删除容器**

```
docker-compose rm
```

上面的命令可以将已经停止运行的容器进行删除，也可以将停止和删除用一条命令代替

```
docker-compose down
```

更多命令请执行 docker-compose -h 查看。