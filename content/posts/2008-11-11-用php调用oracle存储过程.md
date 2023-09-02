---
title: 用PHP调用Oracle存储过程
author: admin
type: post
date: 2008-11-11T05:21:46+00:00
excerpt: |
 PHP程序访问数据库，完全可以使用存储过程，有人认为使用存储过程便于维护,不过仁者见仁，智者见智，在这个问题上，偶认为使用存储过程意味着必须要DBA和开发人员更紧密配合,如果其中一方更变，则显然难以维护。

 但是使用存储过程至少有两个最明显的优点：速度和效率。使用存储过程的速度显然更快。
 在效率上，如果应用一次需要做一系列SQL操作，则需要往返于PHP与ORACLE，不如把该应用直接放到数据库方以减少往返次数，增加效率。
 但是在INTERNET应用上，速度是极度重要的，所以很有必要使用存储过程。
url: /archives/532
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php

---
   PHP程序访问数据库，完全可以使用存储过程，有人认为使用存储过程便于维护,不过仁者见仁，智者见智，在这个问题上，偶认为使用存储过程意味着必须要DBA和开发人员更紧密配合,如果其中一方更变，则显然难以维护。

    但是使用存储过程至少有两个最明显的优点：速度和效率。使用存储过程的速度显然更快。
    在效率上，如果应用一次需要做一系列SQL操作，则需要往返于PHP与ORACLE，不如把该应用直接放到数据库方以减少往返次数，增加效率。
但是在INTERNET应用上，速度是极度重要的，所以很有必要使用存储过程。
偶也是使用PHP调用存储过程不久，做了下面这个列子。

代码:-

//建立一个TEST表
CREATE TABLE TEST (
  ID        NUMBER(16)        NOT NULL,
  NAME      VARCHAR2(30)      NOT NULL,
  PRIMARY KEY (ID)
);

//插入一条数据
INSERT INTO TEST VALUES (5, ‘PHP_BOOK’);

//建立一个存储过程
CREATE OR REPLACE PROCEDURE PROC_TEST (
  p_id IN OUT NUMBER,
  p_name OUT VARCHAR2
) AS
BEGIN
  SELECT NAME INTO p_name
    FROM TEST
    WHERE ID = 5;
END PROC_TEST;
/

PHP代码:——————-

”;

?>