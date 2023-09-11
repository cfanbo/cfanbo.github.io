---
title: Mysql中出现的＂MySQL Got error 139 from storage engine＂的原因
author: admin
type: post
date: 2012-05-08T16:49:52+00:00
url: /archives/12886
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---
今天再从excel里导入数据到Mysql中的时候，发现一个表里的数据总是导入失败．后来经过查找是用的INNODB表引擎的问题．而换成MYISAM表引擎的话，则不存在此问题．并试图先导入成Myisam表，然后手动修改表引擎为INNODB.结果提示＂MySQL Got error 139 from storage engine＂错误．经google一番发现．是由于INNODB单条记录有8K的限制，而导入的excel表里字段不到20个．内容特别的多的．

官方解释如下：

Solution:

1.divide your table into small ones. If one table contain more than 10 text colums, and the data contain is a little bit long. this error will be thrown out.

2.modify InnoDB to MyISAM.Problem description:


```
Description:
Since upgrading MySQL from version 4.0.xx to 4.1.11, when trying to place data into a
record it can fail with the error "#1030 - Got error 139 from storage engine". The problem
appears to be if the data reaches a certain size. I first found the problem when trying to
import a dump created by phpMyAdmin for one of my tables.

I am using the "Standard" install package from MySQL Web site. Running Apple Mac/MacOS X
10.3.9. PHP is version 4.3.10 (client API is 3.23.49).

I use phpMyAdmin to administer the DB, but I have run tests using PHP and the problem
still occurs.

I have also performed tests on my ISP's server (UNIX based) and the problem also occurs,
they are using MySQL client API of 4.1.11.

I still have access to a 4.0 server and can import the dump with no problems at all, even
using a dump from MySQL 4.1.11 using the "backward compatibility" option in phpMyAdmin.

How to repeat:
Create an InnoDB table with 1 INT field and 11 TEXT fields.

Create an index on the INT field of type "PRIMARY".

In the first 10 text fields, enter as many "a" characters as phpMyAdmin will allow. This
is 32000 characters in each field. Leave the last text field empty. Click "Go" to save the
changes. This should save without any problems.

Then copy/paste field 10 to field 11 and click "Go". This results in a "#1030 - Got error
139 from storage engine" error.

If I remove the characters little by little at some point it will work, suggesting a size
problem.

Please let me know if you need any additional information.
```

[21 Apr 2005 7:02] Heikki Tuuri


```
Hi!

In 4.1, to support at least 256-character UTF-8 column prefix indexes, InnoDB stores at
least 768 bytes of each column 'internally' to the record. With 11 TEXT fields you will
run over the 8000 byte record len limit.

./include/dict0mem.h:159:#define DICT_MAX_COL_PREFIX_LEN        768

I probably need to update the manual.

Thank you,

Heikki
```

[21 Apr 2005 10:34] Andrew Blee


```
Hi Heikki

Many thanks for the reply.

I read what you wrote several times, but a lot of it still went over my head, so apologies
if what I  write below is irrelevant :-)

Is this 8000 byte limit you mention the same one mentioned in section 15.17 of the manual,
i.e. because of the 16k database page size?

After reading the manual, section 15.17, I know:

1. The Database Page Size is set as standard as 16k - server needs recompiling to use a
new limit. From what I can see this setting affects the maximum row length for NON TEXT
and BLOB fields. If I set this limit to 64k I would expect to get a maximum of 41 TEXT
columns before this error occured?

2. Maximum Row Length for TEXT and BLOB columns is not mentioned, but 4GB limit is
mentioned for LONGTEXT and LONGBLOB columns.

3. Total Row Length cannot exceed 4GB.

4. "The internal maximum key length is 3500 bytes, but MySQL itself restricts this to 1024
bytes." - Not sure what this means :-)

You mentioned "column prefix indexes", not sure if the index I created is the same thing
so I removed this as well as the INT field and the problem still occurs. I.e. I had a
record with 11 TEXT fields only.

As a standard build, I find it hard to believe MySQL/InnoDB cannot support 11 TEXT fields
in one table. An 8000 byte limit for "internal" information about column prefix indexes
seems very small.

Are there any links I can read to help my confusion about what limits apply and when. More
importantly what changes can I make to help me solve this problem.

Many thanks for your time.
```

另外一篇介绍文章：