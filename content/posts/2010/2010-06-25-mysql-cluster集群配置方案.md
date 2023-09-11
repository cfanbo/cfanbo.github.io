---
title: MySQL Cluster集群配置方案
author: admin
type: post
date: 2010-06-25T13:33:43+00:00
url: /archives/3945
IM_contentdowned:
 - 1
categories:
 - MySQL
 - 前端设计
tags:
 - 集群
 - mysql

---
在为某证券公司设计其OA架构时，初期客户是30万用户在线；然而在项目实施中，客户又提出50万用户同时在线的需求，而且都有写的需求；这样初始的设计 master-master-slave，读写分离满足不了客户的要求，所以我们打算采用Mysql Cluster方案；MySQL Cluster 是MySQL适合于分布式计算环境的高实用、高冗余版本。它采用了NDB Cluster 存储引擎，允许在1个Cluster中运行多个MySQL服务器。在MyQL 5.0及以上的二进制版本中、以及与最新的Linux版本兼容的RPM中提供了该存储引擎。

一、MySQL Cluster概述

MySQL Cluster 是一种技术，该技术允许在无共享的系统中部署“内存中”数据库的 Cluster 。通过无共享体系结构，系统能够使用廉价的硬件，而且对软硬件无特殊要求。此外，由于每个组件有自己的内存和磁盘，不存在单点故障。

MySQL Cluster 由一组计算机构成，每台计算机上均运行着多种进程，包括MySQL服务器，NDB Cluster 的数据节点，管理服务器，以及（可能）专门的数据访问程序。

所有的这些节点构成一个完成的MySQL集群体系。数据保存在“NDB存储服务器”的存储引擎中，表（结构）则保存在“MySQL服务器”中。应用程序通 过“MySQL服务器”访问这些数据表，集群管理服务器通过管理工具(ndb_mgmd)来管理“NDB存储服务器”。

通过将MySQL Cluster 引入开放源码世界，MySQL为所有需要它的人员提供了具有高可用性、高性能和可缩放性的 Cluster 数据管理。

二、MySQL Cluster 基本概念

“NDB” 是一种“内存中”的存储引擎，它具有可用性高和数据一致性好的特点。

MySQL Cluster 能够使用多种故障切换和负载平衡选项配置NDB存储引擎，但在 Cluster 级别上的存储引擎上做这个最简单。MySQL Cluster的NDB存储引擎包含完整的数据集，仅取决于 Cluster本身内的其他数据。

目前，MySQL Cluster的 Cluster部分可独立于MySQL服务器进行配置。在MySQL Cluster中， Cluster的每个部分被视为1个节点。

管理(MGM)节点：这类节点的作用是管理MySQL Cluster内的其他节点，如提供配置数据、启动并停止节点、运行备份等。由于这类节点负责管理其他节点的配置，应在启动其他节点之前首先启动这类节 点。MGM节点是用命令“ndb_mgmd”启动的。

数据节点：这类节点用于保存 Cluster的数据。数据节点的数目与副本的数目相关，是片段的倍数。例如，对于两个副本，每个副本有两个片段，那么就有4个数据节点。不过没有必要设 置多个副本。数据节点是用命令“ndbd”启动的。

SQL节点：这是用来访问 Cluster数据的节点。对于MySQL Cluster，客户端节点是使用NDB Cluster存储引擎的传统MySQL服务器。通常，SQL节点是使用命令“mysqld –ndbcluster”启动的，或将“ndbcluster”添加到“my.cnf”后使用“mysqld”启动。

注释：在很多情况下，术语“节点”用于指计算机，但在讨论MySQL Cluster时，它表示的是进程。在单台计算机上可以有任意数目的节点，为此，我们采用术语“ Cluster主机”。

管理服务器(MGM节点)负责管理 Cluster配置文件和 Cluster日志。 Cluster中的每个节点从管理服务器检索配置数据，并请求确定管理服务器所在位置的方式。当数据节点内出现新的事件时，节点将关于这类事件的信息传输 到管理服务器，然后，将这类信息写入 Cluster日志。

此外，可以有任意数目的 Cluster客户端进程或应用程序。它们分为两种类型：
标准MySQL客户端：对于MySQL Cluster，它们与标准的（非 Cluster类）MySQL没有区别。换句话讲，能够从用PHP、Perl、C、C++、Java、Python、Ruby等编写的现有MySQL应用 程序访问MySQL Cluster。

管理客户端：这类客户端与管理服务器相连，并提供了启动和停止节点、启动和停止消息跟踪（仅调试版本）、显示节点版本和状态、启动和停止备份等的命令。

三、开始准备

1、准备服务器

现在，我们计划建立有5个节点的MySQL CLuster体系，因此需要用到5台机器，分别做如下用途：

用途 主机名 IP
管理节点 ndb_mgmd 192.168.5.101
数据节点1 ndb1 192.168.5.102
数据节点2 ndb2 192.168.5.103
sql节点1 mysqld1 192.168.5.104
sql节点2 mysqld2 192.168.5.105

2、注意事项及其他

每个节点的操作系统都是Linux，下面的描述中将使用主机名，不再使用IP地址来表示。由于MySQL Cluster采用TCP/IP方式连接，并且节点之间的数据传输没有加密，因此这个体系最好只在单独的子网中运行，并且考虑到传输的速率，强烈建议不要 跨越公网使用这个体系。为了测试环境的准备性，我这里所有hostname相关我全部写成ip，所需软件我推荐用mysql-max-5.0.24，下载 地址为(已测) [http://www.filewatcher.com/_/?q=mysql-max-5.0.24-linux-i686.tar.gz](http://www.filewatcher.com/_/?q=mysql-max-5.0.24-linux-i686.tar.gz)

四、开始安装

1、假定条件

在每个节点计算机上都采用mysql 用户来运行Cluster，因此执行如下命令添加相关用户（如果已经存在则略过，且用root用户执行）：

root# /usr/sbin/groupadd mysql

root# /usr/sbin/useradd -g mysql mysql

假设已经下载了mysql可直接使用的二进制安装包，且放在 /root 下了。

2、SQL节点和存储节点(NDB节点)安装(即4个机器重复执行以下步骤)

root# cd /root/

root# tar zxf mysql-max-5.0.24-linux-i686.tar.gz

root# mv mysql-max-5.0.24-linux-i686 /usr/local/mysql/

root# cd /usr/local/mysql/

root# ./configure –prefix=/usr/local/mysql

root# ./scripts/mysql\_install\_db

root# chown -R mysql:mysql /usr/local/mysql/