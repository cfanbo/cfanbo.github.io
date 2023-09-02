---
title: 在sql server中改变表的所有者
author: admin
type: post
date: 2007-08-07T02:00:41+00:00
url: /archives/79
IM_contentdowned:
 - 1
categories:
 - 数据库

---
        在对数据库进行移植的时候，经常会发现表的所有者发生了改变，而造成数据表拒绝访问，我们可以通过下面的语句来修改数据表的所有者：

EXEC sp_changeobjectowner ‘super.article’, ‘dbo’

这个我们就可以dbo的身份对数据库中的表进行相应的操作了！