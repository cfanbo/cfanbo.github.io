---
title: char和varchar2的区别
author: admin
type: post
date: 2008-11-11T05:26:59+00:00
excerpt: |
 |
 区别：
 1．CHAR的长度是固定的，而VARCHAR2的长度是可以变化的， 比如，存储字符串“abc"，对于CHAR (20)，表示你存储的字符将占20个字节(包括17个空字符)，而同样的VARCHAR2 (20)则只占用3个字节的长度，20只是最大值，当你存储的字符小于20时，按实际长度存储。
 2．CHAR的效率比VARCHAR2的效率稍高。
 3．目前VARCHAR是VARCHAR2的同义词。工业标准的VARCHAR类型可以存储空字符串，但是oracle不这样做，尽管它保留以后这样做的权利。Oracle自己开发了一个数据类型VARCHAR2，这个类型不是一个标准的VARCHAR，它将在数据库中varchar列可以存储空字符串的特性改为存储NULL值。如果你想有向后兼容的能力，Oracle建议使用VARCHAR2而不是VARCHAR。
url: /archives/534
IM_contentdowned:
 - 1
categories:
 - 数据库

---
区别：
1．CHAR的长度是固定的，而VARCHAR2的长度是可以变化的， 比如，存储字符串“abc”，对于CHAR (20)，表示你存储的字符将占20个字节(包括17个空字符)，而同样的VARCHAR2 (20)则只占用3个字节的长度，20只是最大值，当你存储的字符小于20时，按实际长度存储。
2．CHAR的效率比VARCHAR2的效率稍高。
3．目前VARCHAR是VARCHAR2的同义词。工业标准的VARCHAR类型可以存储空字符串，但是oracle不这样做，尽管它保留以后这样做的权利。Oracle自己开发了一个数据类型VARCHAR2，这个类型不是一个标准的VARCHAR，它将在数据库中varchar列可以存储空字符串的特性改为存储NULL值。如果你想有向后兼容的能力，Oracle建议使用VARCHAR2而不是VARCHAR。

何时该用CHAR，何时该用varchar2？
           CHAR与VARCHAR2是一对矛盾的统一体，两者是互补的关系.
VARCHAR2比CHAR节省空间，在效率上比CHAR会稍微差一些，即要想获得效率，就必须牺牲一定的空间，这也就是我们在数据库设计上常说的‘以空间换效率’。
   VARCHAR2虽然比CHAR节省空间，但是如果一个VARCHAR2列经常被修改，而且每次被修改的数据的长度不同，这会引起‘行迁移’(Row Migration)现象，而这造成多余的I/O，是数据库设计和调整中要尽力避免的，在这种情况下用CHAR代替VARCHAR2会更好一些。