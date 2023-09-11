---
title: 'LVS & MySQL NDB Cluster'
author: admin
type: post
date: 2010-06-26T17:04:50+00:00
url: /archives/4117
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - 系统架构
 - 网站架构
 - mysql

---
章文嵩博士（LVS开源项目创始人）进入淘宝好几个月了，今天是他第一次讲解LVS的实现原理。作为DBA的一员，终于近距离膜拜了大牛。
讲解的内容就不具体介绍了，在[LVS 官方网站][1]上面可以找到。PPT的内容和网站上基本上一样，只是讲解人是章博士本人。我在这整理一下自己的理解，不对请大家指正。 ^_^

组成LVS最重要的部分有三个：请求分发服务器、处理服务器、共享存储。

典型的Web集群并不需要共享存储，只有请求分发服务器和处理服务器，如下图所示：
[![](https://blogstatic.haohtml.com//uploads/2023/09/LVS_WEB-300x245.jpg)][2]
如果完成请求需要基于数据，那么共享存储就是LVS必须的组件了。LVS邮件服务器集群如下所示：
[![](https://blogstatic.haohtml.com//uploads/2023/09/LVS_MAIL.jpg)][3]
目前能应用于LVS的MySQL集群只能是NDB Cluster，因为MySQL众多的存储引擎中，只有NDB Cluster实现了共享存储的功能。
在NDB Cluster中，SQL Node相当于处理服务器，Data Node相当于共享存储。LVS可以让应用程序的开发更加简单，开发人员并不需要知道执行SQL的数据库服务器到底是哪一个，但是可以获得自己想要的数 据。而NDB Cluster提供的数据拆分和扩容功能，保证了数据库的可扩展性。

美中不足的是，NDB Cluster的性能实在太差。即使LVS负载均衡做得很好很强大，NDB Cluster也会成为瓶颈。
根据我的观察发现，LVS的架构非常适用于CPU-bound的应用场景。虽然共享存储的引入使得LVS能够支持有状态的连接，但是LVS并不适用于 IO-bound的应用。因为不管负载如何均衡，最终瓶颈在于共享存储之上。而共享存储如何拓展，LVS并不关心。也许只有NDB Cluster提升了IO性能之后，LVS才能真正在MySQL方面应用起来。

非常感谢章博士的分享，NDB Cluster终于让我觉得不那么鸡肋了。

[1]: http://www.linuxvirtualserver.org/whatis.html
