---
title: MySQL 集群在Server1与Server2上如何安装MySQL
author: admin
type: post
date: 2010-06-25T13:36:45+00:00
url: /archives/3951
IM_contentdowned:
 - 1
categories:
 - MySQL
 - 前端设计
tags:
 - 集群
 - mysql

---
我们今天主要向大家介绍的是MySQL 集群，其中包括对MySQL 集群的概念介绍，以及如何在Server1与Server2上正确对MySQL进行安装 ，还有对安装与配置管理节点服务器(Server3)的正确操作 ，配置集群服务器并启动MySQL 。

**一、介绍**

这篇文档旨在介绍如何安装配置基于2台服务器的MySQL集群。并且实现任意一台服务器出现问题或宕机时MySQL依然能够继续运行。

注意！

虽然这是基于2台服务器的MySQL集群，但也必须有额外的第三台服务器作为管理节点，但这台服务器可以在集群启动完成后关闭。同时需要注意的是并 不推荐在集群启动完成后关闭作为管理节点的服务器。尽管理论上可以建立基于只有2台服务器的MySQL集群，但是这样的架构，一旦一台服务器宕机之后集群 就无法继续正常工作了，这样也就失去了集群的意义了。出于这个原因，就需要有第三台服务器作为管理节点运行。

另外，可能很多朋友都没有3台服务器的实际环境，可以考虑在VMWare或其他虚拟机中进行实验。

下面假设这3台服务的情况：

Server1: MySQL1.vmtest.net 192.168.0.1

Server2: MySQL2.vmtest.net 192.168.0.2

Server3: MySQL3.vmtest.net 192.168.0.3

Servers1和Server2作为实际配置MySQL集群的服务器。对于作为管理节点的Server3则要求较低，只需对Server3的系统 进行很小的调整并且无需安装MySQL，Server3可以使用一台配置较低的计算机并且可以在Server3同时运行其他服务。

**二、在Server1和Server2上安装MySQL**

从

注意：必须是max版本的MySQL，Standard版本不支持集群部署！

以下步骤需要在Server1和Server2上各做一次

```


    # mv MySQL-max-4.1.9-pc-linux-gnu-i686.tar.gz /usr/local/




    # cd /usr/local/




    # groupadd MySQL




    # useradd -g MySQL MySQL




    # tar -zxvf MySQL-max-4.1.9-pc-linux-gnu-i686.tar.gz




    # rm -f MySQL-max-4.1.9-pc-linux-gnu-i686.tar.gz




    # mv MySQL-max-4.1.9-pc-linux-gnu-i686 MySQL




    # cd MySQL




    # scripts/MySQL_install_db --user=MySQL




    # chown -R root .




    # chown -R MySQL data




    # chgrp -R MySQL .




    # cp support-files/MySQL.server /etc/rc.d/init.d/MySQLd




    # chmod x /etc/rc.d/init.d/MySQLd




    # chkconfig --add MySQLd



```

此时不要启动MySQL！

**三、安装并配置管理节点服务器(Server3)**

作为管理节点服务器，Server3需要ndb\_mgm和ndb\_mgmd两个文件：

```


    # mkdir /usr/src/MySQL-mgm




    # cd /usr/src/MySQL-mgm




    # tar -zxvf MySQL-max-4.1.9-pc-linux-gnu-i686.tar.gz




    # rm MySQL-max-4.1.9-pc-linux-gnu-i686.tar.gz




    # cd MySQL-max-4.1.9-pc-linux-gnu-i686




    # mv bin/ndb_mgm .




    # mv bin/ndb_mgmd .




    # chmod x ndb_mg*




    # mv ndb_mg* /usr/bin/




    # cd




    # rm -rf /usr/src/MySQL-mgm



```

现在开始为这台管理节点服务器建立配置文件：

```


    # mkdir /var/lib/MySQL-cluster




    # cd /var/lib/MySQL-cluster




    # vi config.ini



```

在config.ini中添加如下内容：

```


    [NDBD DEFAULT]




    NoOfReplicas=2




    [MySQLD DEFAULT]




    [NDB_MGMD DEFAULT]




    [TCP DEFAULT]




    # Managment Server




    [NDB_MGMD]



```

HostName=192.168.0.3 #管理节点服务器Server3的IP地址

\# Storage Engines

[NDBD]

HostName=192.168.0.1 #MySQL集群Server1的IP地址

DataDir= /var/lib/MySQL-cluster

[NDBD]

HostName=192.168.0.2 #MySQL集群Server2的IP地址

DataDir=/var/lib/MySQL-cluster

\# 以下2个[MySQLD]可以填写Server1和Server2的主机名。

\# 但为了能够更快的更换集群中的服务器，推荐留空，否则更换服务器后必须对这个配置进行更改。

[MySQLD]

[MySQLD]

保存退出后，启动管理节点服务器Server3：

\# ndb_mgmd

启动管理节点后应该注意，这只是管理节点服务，并不是管理终端。因而你看不到任何关于启动后的输出信息。

**四、配置集群服务器并启动MySQL**

在Server1和Server2中都需要进行如下改动：

\# vi /etc/my.cnf

[MySQLd]

ndbcluster

ndb-connectstring=192.168.0.3 #Server3的IP地址

[MySQL_cluster]

ndb-connectstring=192.168.0.3 #Server3的IP地址

保存退出后，建立数据目录并启动MySQL：

```


    # mkdir /var/lib/MySQL-cluster




    # cd /var/lib/MySQL-cluster




    # /usr/local/MySQL/bin/ndbd --initial




    # /etc/rc.d/init.d/MySQLd start



```

可以把/usr/local/MySQL/bin/ndbd加到/etc/rc.local中实现开机启动。

注意：只有在第一次启动ndbd时或者对Server3的config.ini进行改动后才需要使用–initial参数！

以上的相关内容就是对MySQL 集群的部分内容介绍，望你能有所收获。