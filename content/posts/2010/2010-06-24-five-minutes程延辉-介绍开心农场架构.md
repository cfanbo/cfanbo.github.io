---
title: Five Minutes程延辉 介绍开心农场架构
author: admin
type: post
date: 2010-06-24T13:49:58+00:00
url: /archives/3907
IM_data:
 - 'a:2:{s:66:"/upload/attachment/119984/6088cf98-e268-334c-b9f7-3ce4da338e14.jpg";s:66:"/upload/attachment/119984/6088cf98-e268-334c-b9f7-3ce4da338e14.jpg";s:66:"/upload/attachment/119986/4ef0fd22-516a-3742-90f9-dba9fc6895bb.jpg";s:66:"/upload/attachment/119986/4ef0fd22-516a-3742-90f9-dba9fc6895bb.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 程序开发
 - 前端设计

---

Five Minutes 公司程延辉（小名康天） 介绍开心农场架构，social game的技术挑战，支持千万级DAU的social game技术架构。这是一个对于开发者来说，非常精彩，非常有实用性指导的一次演讲，详细介绍了很多技术内幕。


[>>猛击这里下载演讲ppt<<](http://www.javaeye.com/topics/download/e4f72cf2-2baa-3f4b-9578-d220b6bceea9)

Five Minutes 公司的著名social game 开心农场，目前非常受用户欢迎，包括国外的Facebook，国内的开心网都是如此，是全球最大的social game，台下热烈掌声。呵呵。开心农场这个游戏从介绍看，相当成功，最早是08年9月在xiaonei上线，而后在51等平台推广，包括Facebook。现在已经有1570万游戏用户了，其中包括50万的Facebook用户。


开心农场架构主要难点：1。如何存储大规模的用户数据千万级2。如果应对大量访问每天数亿请求量3。如果应对数据的频繁修改，每秒数万次数据修改。


解决的方式


优化：


1。负载均衡，web服务器平行扩展。


2。服务器性能优化。


3。异步处理，缓存数据接口，Linux内核参数优化，挖掘PHP的效率，用fastcgi模式运行php，用EAccelerator加速。固定不变数据做成php配置文件，用C开发PHP扩展等。


数据库性能优化：


1。数据库分库分表，所有数据全部设计成 key－》value形式，不用join。


2。使用INNODB，经常操作的数据表中所有字段尽量设计成数值型，用update替代INSERT和DELETE操作


异步处理：整个系统最关键的部分，


原则：把客户端暂时不需要的数据进行异步处理。


实例：讲非核心数据先写入memcached，异步更新到数据库，合并数据库更新操作，Feed和Notification的异步发送。


利用客户端资源：Flash屏蔽重复操作和不必要请求，Flash进行一些计算减轻服务器的复旦，例如好友排序等。Flash缓存一些数据。


social game ＝ social ＋ game。实时互动（大负载）和非实时互动（大负载）。


服务器角色：场景服务器，逻辑服务器，admin服务器，gateway，架构逻辑还是挺复杂的，每天处理亿级请求的架构，完全和百万级不一样！完全能够通过平行扩展的方式应对，gateway和场景服务器都完全可以增加。


Blue Whale是他们们正在开发的解决长连接的social game架构。


开心农场在现场招聘：需要C++,Python， Flash AS3程序员。


程延辉的演讲获得了在场热烈的掌声。


[![](http://blog.haohtml.com/wp-content/uploads/2010/06/301jh54033.jpg)](/wp-content/uploads/2010/06/301jh54033.jpg)

![](/upload/attachment/119984/6088cf98-e268-334c-b9f7-3ce4da338e14.jpg)

![](/upload/attachment/119986/4ef0fd22-516a-3742-90f9-dba9fc6895bb.jpg)