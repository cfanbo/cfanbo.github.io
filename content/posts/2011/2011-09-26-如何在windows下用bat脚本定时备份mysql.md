---
title: 如何在windows下用bat脚本定时备份mysql
author: admin
type: post
date: 2011-09-26T02:01:10+00:00
url: /archives/11519
IM_data:
 - 'a:1:{s:41:"http://imysql.cn/files/pictures/email.gif";s:65:"http://blog.haohtml.com/wp-content/uploads/2011/09/2986_email.gif";}'
IM_contentdowned:
 - 1
categories:
 - MySQL
tags:
 - mysql

---
作/译者：叶金荣（Email: ![](http://imysql.cn/files/pictures/email.gif)），来源：[http://imysql.cn][1]，转载请注明作/译者和出处，并且不能用于商业用途，违者必究。

并不是所有MySQL都运行在Linux下，windows下也需要做例行备份，下面是用bat脚本做自动化备份的例子，大家可以参考下。

```
rem
rem C:\Program Files\WinRAR 需要放到 path 下，才能调用rar cli工具
rem
rem 跳转到工作目录下
f:
cd f:\DBBAK
rem 设置变量：备份文件名
SET BAK_FILE=MY_DBBAK_%date:~0,-4%.sql
rem 设置变量：日志文件名
SET LOG_FILE=MY_DBBAK.log
rem 记录日志
echo "%date%" >> %LOG_FILE%
rem 开始做备份
mysqldump --default-character-set=utf8 -hlocalhost -uroot -R --triggers --single-transaction -B mydb > %BAK_FILE%
rem 压缩备份文件
rar a %BAK_FILE%.rar %BAK_FILE%
rem 删除源文件
del /F %BAK_FILE%
echo "%date%" >> %LOG_FILE%
echo "" >> %LOG_FILE%
```

部署完脚本后，剩下的就是在系统中添加“计划任务”项目了。

 [1]: http://imysql.cn/