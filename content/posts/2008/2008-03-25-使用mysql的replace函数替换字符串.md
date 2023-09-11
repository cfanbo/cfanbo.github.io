---
title: 使用mysql的replace函数替换字符串
author: admin
type: post
date: 2008-03-25T12:36:54+00:00
url: /archives/280
IM_contentdowned:
 - 1
categories:
 - 数据库

---
一直以为只有mysql才有replace这个方法，后来一查，sql server居然也有，看来自己真是无知啊。。。。

比如你要将 表 tb1里面的 f1字段的abc替换为def

Update tb1 SET f1=REPLACE(f1, ‘abc’, ‘def’);

REPLACE(str,from\_str,to\_str)
在字符串   str   中所有出现的字符串   from\_str   均被   to\_str替换，然后返回这个字符串：
mysql>   Select   REPLACE(‘www.mysql.com’,   ‘w’,   ‘Ww’);
                  ->   ‘WwWwWw.mysql.com’
这个函数是多字节安全的。

示例：
Update  \`dede_addonarticle\`  SET body =  REPLACE ( body,
 ‘’,
 ” );
Update  \`dede_addonarticle\`  SET body =  REPLACE ( body,
 ‘’,
 ” );
Update  \`dede_addonarticle\`  SET body =  REPLACE ( body,
 ‘’,
 ” );
Update  \`dede_archives\`  SET title=  REPLACE ( title,
 ‘大洋新闻 – ‘,
 ” );
Update  \`dede_addonarticle\`  SET body =  REPLACE ( body,
 ‘../../../../../../’,
 ‘http://special.dayoo.com/meal/’ );

mysql replace

用法1.replace intoreplace into table (id,name) values(‘1‘,‘aa‘),(‘2‘,‘bb‘)
此语句的作用是向表table中插入两条记录。
2.replace(object, search,replace)
把object中出现search的全部替换为replaceselect replace(‘www.163.com‘,‘w‘,‘Ww‘)—>WwW wWw.163.com

例：把表table中的name字段中的 aa替换为bbupdate table set name=replace(name,‘aa‘,‘bb‘)