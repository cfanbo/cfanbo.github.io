---
title: 基于docker环境实现Elasticsearch 集群环境
author: admin
type: post
date: 2019-07-12T07:19:16+00:00
url: /archives/19059
categories:
 - 系统架构
tags:
 - es

---
最近搭建了es集群的时候，现在需要测试添加一个新的数据节点，项目是使用docker-compose命令来搭建的。

以下基于最新版本 es7.2.0进行, 配置文件目录为 es, 所以docker 在创建网络的时候，网络名称会以 es_ 前缀开始，如本例中我们在docker-composer.yaml文件中指定了网络名称为esnet,但docker生成的实例名称为 es_esnet，至于网络相关的信息可以通过 `docker network --help` 查看。

## 搭建es集群 

// docker-compose.yaml 集群配置文件

```
version: '2.2'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.2.0
    container_name: es01
    environment:
      - node.name=es01
      - node.master=true
      - node.data=true
      - discovery.seed_hosts=es02
      - cluster.initial_master_nodes=es01,es02
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - esnet
  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.2.0
    container_name: es02
    environment:
      - node.name=es02
      - discovery.seed_hosts=es01
      - cluster.initial_master_nodes=es01,es02
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata02:/usr/share/elasticsearch/data
    networks:
      - esnet
  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.2.0
    container_name: es03
    environment:
      - node.name=es03
      - discovery.seed_hosts=es01
      - cluster.initial_master_nodes=es01,es02
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata03:/usr/share/elasticsearch/data
    networks:
      - esnet
volumes:
  esdata01:
    driver: local
  esdata02:
    driver: local
  esdata03:
    driver: local

networks:
  esnet:

```

集群配置了3个master节点，并同时作为数据节点使用，当节点未指定 node.master和node.data的时候，默认值为 true 。执行命令

```
$ docker-compose up
```

启动集群。

验证集群环境是否搭建成功，在浏览器里访问  和  显示正常。三个节点角色均为mdi

```
ip           heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
192.168.16.3           28          74   3    0.14    0.94     4.97 mdi       *      es02
192.168.16.4           28          74   3    0.14    0.94     4.97 mdi       -      es01
192.168.16.2           25          74   3    0.14    0.94     4.97 mdi       -      es03
```

## 添加集群新的节点 

添加es数据节点文件 join-docker-compose.yaml

```
version: '2.2'
services:
  es04:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.2.0
    container_name: es04
    environment:
      - node.name=es04
      - node.master=false
      - node.data=true
      - cluster.initial_master_nodes=es01,es02
      - cluster.name=docker-cluster
      - discovery.seed_hosts=es01
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata04:/usr/share/elasticsearch/data
    networks:
      - esnet


volumes:
  esdata04:
    driver: local

networks:
  esnet:
    external:
      name: es_esnet

```

这里指定了 `node.master=false`，执行命令

```
$ docker-compose -f join-docker-compose.yaml up
```

注意这里手动指定了 yaml 文件，两个配置文件都在同一个es目录里。

再次使用上面的  进行验证

```
ip           heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
192.168.16.3           26          93  19    0.44    0.28     1.30 mdi       *      es02
192.168.16.5           22          93  14    0.44    0.28     1.30 di        -      es04
192.168.16.4           15          93  18    0.44    0.28     1.30 mdi       -      es01
192.168.16.2           37          93  20    0.44    0.28     1.30 mdi       -      es03
```

这里我们可以看到新增加的节点 es04,节点角色为di, 这个节点由于指定了 `node.master=false` 说明此节点并不参与master leader节点的选举。

集群节点使用的docker网络为 es_esnet

## 测试es集群master的选举（高可用） 

上面我们可以看到当前es02这个master节点为leader，我们现在手动停止这个master容器，让其它的两个master中选举一个leader，执行命令

```
$ docker stop es02
```

此时，再用上面的方法查看一下集群节点情况

```
ip           heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
192.168.16.5           40          74   2    0.19    0.27     0.94 di        -      es04
192.168.16.4           15          74   2    0.19    0.27     0.94 mdi       *      es01
192.168.16.2           39          74   2    0.19    0.27     0.94 mdi       -      es03
```

可以看到es02节点消失了，现在的master leader 节点为 es01。如果我们再把容器启动起来的话，发现es02作为了一个普通的master节点加入到了集群。

## 注意事项： 

其实es集群的环境搭建挺容易的，我在搭建过程中遇到了一此坑，花费了好久才算爬出来，下面记录下来供大家参考。

### 一、确认分配给 docker 软件的内存是否足够 

在上一篇文章( ) 里已经写过了，由于docker 分配的内存不足，导致启动一个新的es节点，会直接killed掉原来的es节点，发生OOM的现象，并且这个问题通过直接查看容器日志根本排查不到，容器内并未有相关的日志信息

### 二、保证环境的干净 

使用 docker-compose 命令启动es节点容器成功后，如果需要对旧节点的配置内容进行修改的话，则强烈建议执行以下命令进行容器的清理工作

```
$ docker-compose -f 配置文件.yaml down -v
```

以上命令会将容器及容器关联的数据卷 volume 信息进行一并删除。否则容易出现新启动的节点又单独变成了一个集群，这时会出现跨集群节点加入被拒绝的错误。我在搭建环境的时候，创建用的 docker-compose up 命令，但修改配置文件后，手动执行 “docker rm 容器ID” 将容器删除，再次执行了 docker-compose up命令时，会出现上面说的这个问题，在这个坑里呆了好久才算出来。如果一定要想用docker rm 命令删除容器的话，添加添加-f参数，将volume 一并删除，如 `docker rm -f es03`。

### 三、参数 discovery.zen.minimum\_master\_nodes 

这里用的是es7.2.0的版本，服务启动时提示参数项`discovery.zen.minimum_master_nodes` 在下一个版本中即将废除的，但在官方文档里没有找到说明信息，这一点待确认。