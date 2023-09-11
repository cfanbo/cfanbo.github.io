---
title: 使用MySQL Proxy和MySQL Replication实现读写分离
author: admin
type: post
date: 2009-05-16T04:20:08+00:00
excerpt: MySQL Replication可以将master的数据复制分布到多个slave上，然后可以利用slave来分担master的读压力。那么对于前台应用来说，就要考虑如何将读的压力分布到多个slave上。如果每个应用都需要来实现读写分离的算法，一则成本太高，二来如果slave增加更多的机器，应用就要随之修改。明显的，如果在应用和数据库间加一个专门用于实现读写分离的中间层，则整个系统的架构拥有更好的扩展性。
url: /archives/1395
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---

[MySQL Replication](http://www.ningoo.net/html/2007/mysql_replication_configuration.html) 可以将master的数据复制分布到多个slave上，然后可以利用slave来分担master的读压力。那么对于前台应用来说，就要考虑如何将读的压力分布到多个slave上。如果每个应用都需要来实现读写分离的算法，一则成本太高，二来如果slave增加更多的机器，应用就要随之修改。明显的，如果在应用和数据库间加一个专门用于实现读写分离的中间层，则整个系统的架构拥有更好的扩展性。


MySQL Proxy就是这么一个中间层代理，简单的说，MySQL Proxy就是一个连接池，负责将前台应用的连接请求转发给后台的数据库，并且通过使用 [lua脚本](http://www.lua.org/)，可以实现复杂的连接控制和过滤，从而实现读写分离和负载平衡。对于应用来说，MySQL Proxy是完全透明的，应用则只需要连接到MySQL Proxy的监听端口即可。当然，这样proxy机器可能成为单点失效，但完全可以使用多个proxy机器做为冗余，在应用服务器的连接池配置中配置到多个proxy的连接参数即可。


安装教程请参考: [http://blog.haohtml.com/archives/9465](http://blog.haohtml.com/archives/9465)

[![](https://blogstatic.haohtml.com//uploads/2023/09/lim98ihh.png)](/wp-content/uploads/2009/05/lim98ihh.png)

[MySQL Proxy](http://forge.mysql.com/wiki/MySQL_Proxy) 是2007年6月由 [Jan Kneschke](http://jan.kneschke.de/) 发布的一个项目，基于 [GPL](http://www.gnu.org/licenses/old-licenses/gpl-2.0.html)，目前还处于Alpha测试阶段，最新版本0.6.1，可以从 [这里下载](http://dev.mysql.com/downloads/mysql-proxy/)。使用MySQL Proxy需要MySQL 5.0.x以上版本，详细的安装使用手册可以参考 [MySQL 5.1手册第27章](http://dev.mysql.com/doc/refman/5.1/en/mysql-proxy.html)。


mysql-proxy –help-all

Usage:

mysql-proxy.exe [OPTION…] – MySQL Proxy

Help Options:

 -?, –help     Show help options

–help-all      Show all help options

–help-admin    Show options for the admin-module

–help-proxy    Show options for the proxy-module


admin module

–admin-address=   listening address:port of internal admin-server (default:4041)


proxy-module

–proxy-address=   listening address:port of the proxy-server (default:4040)

–proxy-read-only-backend-addresses=  address:port of the remote slave-server (default: not set)

–proxy-backend-addresses=   address:port of the remote backend-servers (default: 127.0.0.1:3306)

–proxy-skip-profiling  disables profiling of queries (default: enabled)

–proxy-fix-bug-25371  fix bug #25371 (mysqld > 5.1.12) for older libmysql versions

–proxy-lua-script=   filename of the lua script (default: not set)

–no-proxy     Don’t start proxy-server


Application Options:

 -V, –version      Show version

–daemon        Start in daemon-mode

–pid-file=    PID file in case we are started as daemon