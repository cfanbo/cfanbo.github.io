---
title: 用一个实例讲解oracle数据库中的connect resource权限
author: admin
type: post
date: 2008-11-12T10:15:25+00:00
url: /archives/562
IM_contentdowned:
 - 1
categories:
 - 数据库

---

connect resource权限；

     grant connect,resource to user;

     后用户包括的权限:

**CONNECT角色： –是授予最终用户的典型权利，最基本的**

     ALTER SESSION –修改会话

     CREATE CLUSTER –建立聚簇

     CREATE DATABASE LINK –建立数据库链接

     CREATE SEQUENCE –建立序列

     CREATE SESSION –建立会话

     CREATE SYNONYM –建立同义词

     CREATE VIEW –建立视图

**RESOURCE角色： –是授予开发人员的**

     CREATE CLUSTER –建立聚簇

     CREATE PROCEDURE –建立过程

     CREATE SEQUENCE –建立序列

     CREATE TABLE –建表

     CREATE TRIGGER –建立触发器

     CREATE TYPE –建立类型


**从dba_sys_privs里可以查到:**    SQL> select grantee,privilege from dba_sys_privs

          2 where grantee=’RESOURCE’ order by privilege;

     GRANTEE PRIVILEGE

     ———— ———————-

     RESOURCE CREATE CLUSTER

     RESOURCE CREATE INDEXTYPE

     RESOURCE CREATE OPERATOR

     RESOURCE CREATE PROCEDURE

     RESOURCE CREATE SEQUENCE

     RESOURCE CREATE TABLE

     RESOURCE CREATE TRIGGER

     RESOURCE CREATE TYPE

     已选择8行。