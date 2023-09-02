---
title: mysql中的max_connect_errors
author: admin
type: post
date: 2010-07-02T02:59:06+00:00
url: /archives/4279
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---
连接mysql server出来这个信息

引用

>

> message from server: “Host ‘HP-2B6E9EC1747B’ is blocked because of many connection errors; unblock with ‘mysqladmin flush-hosts'”
>

连接次数失败过多,并超过max\_connect\_erros的值后，服务器会直接拒绝来源机器的所有连接，只要把mysql server默认 max\_connect\_errors = 10
把这个值设置大点就好了，记得一定要执行mysqladmin flush-hosts命令来解锁，原来的主机才可以恢复正常连接的．