---
title: MySQL 性能优化（show processlist中出现大量未认证的连接 建立连接缓慢 unauthenticated user）
author: admin
type: post
date: 2009-06-18T16:14:22+00:00
excerpt: |
 症状：
 MySQL重启后，发现连接非常慢，建立连接后做普通操作还是非常快的，通过Show processlist发现大量unauthenticated user连接

 解决办法：
 MySQL启动参数增加一个skip-name-resolve，即不启用DNS反响解析

 解决过程（推荐学习 :)）

 1. 同一局域网不同机器上连接MySQL服务器，观察响应速度；（非常慢）
 2. 连接建立后做一些简单的操作（非常快 和上面不符，说明不是服务器压力导致的）
 3. 通过tcpdump查看连接的时候详细的响应过程（能看到连接的时候和服务器的多次通讯及所耗费的时间）
 4. 发现正常响应很快，用户名、密码发送完后很长一段时间服务器才给响应
 5. 在服务器上进行同样的连接连接操作，试着分别用IP和机器名去连接看响应速度（看是不是IP解析的时候耗费时间了）
 6. 修改/etc/hosts文件，加入服务器、客户端的IP对应（没有作用 :()
 7. 增加启动参数skip-name-resolve速度立即加快了（问题解决）
url: /archives/1859
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---
症状：
MySQL重启后，发现连接非常慢，建立连接后做普通操作还是非常快的，通过Show processlist发现大量unauthenticated user连接

解决办法：
MySQL启动参数增加一个skip-name-resolve，即不启用DNS反响解析

解决过程（推荐学习 :)）

1. 同一局域网不同机器上连接MySQL服务器，观察响应速度；（非常慢）
2. 连接建立后做一些简单的操作（非常快 和上面不符，说明不是服务器压力导致的）
3. 通过tcpdump查看连接的时候详细的响应过程（能看到连接的时候和服务器的多次通讯及所耗费的时间）
4. 发现正常响应很快，用户名、密码发送完后很长一段时间服务器才给响应
5. 在服务器上进行同样的连接连接操作，试着分别用IP和机器名去连接看响应速度（看是不是IP解析的时候耗费时间了）
6. 修改/etc/hosts文件，加入服务器、客户端的IP对应（没有作用 :()
7. 增加启动参数skip-name-resolve速度立即加快了（问题解决）

原因：
MySQL的认证实际上是user+host的形式（也就是说user可以相同），所以MySQL在处理新连接时会试着去解析客户端连接的IP（查呀查，查不到就算了，可是时间浪费了啊），启用参数skip-name-resolve后MySQL授权的时候就只能用纯IP的形式了，MySQL在这里处理的有点弱智，通过IP连过来的查什么查，又查不到:)
如果用的localhost连接的话,一旦添加上了上面的参数,将出现拒绝ip访问的错误信息!