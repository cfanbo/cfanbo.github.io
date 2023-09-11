---
title: Linux中让crontab以秒为单位执行任务
author: admin
type: post
date: 2014-05-28T04:07:11+00:00
url: /archives/14973
categories:
 - 程序开发
tags:
 - crontab

---
Linux下实现秒级定时任务的两种方案（Crontab 每秒运行）:

**第一种方案，当然是写一个后台运行的脚本一直循环，然后每次循环sleep一段时间。**

```
#!/bin/bash
while true ;do

command

sleep XX //间隔秒数

done
```

或者使用for语句

```
#!/bin/bash

for((i=1;i<=20;i++));do
/home/somedir/scripts.sh > /dev/null 2>&1
sleep 3
done
```

**第二种方案，使用crontab。**

我们都知道crontab的粒度最小是到分钟，但是我们还是可以通过变通的方法做到隔多少秒运行一次。

以下方法将每20秒执行一次

```
crontab -e
* * * * * /bin/date
* * * * * sleep 20; /bin/date.sh
* * * * * sleep 40; /bin/date.sh
```

说明：需要将/bin/date.sh更换成你的命令即可

这种做法去处理隔几十秒的定时任务还好，要是每1秒运行一次就得添加60条记录。。。如果每秒运行还是用方案一吧。
更多crontab命令，请参考： [http://linuxtools-rst.readthedocs.org/zh_CN/latest/tool/crontab.html](http://linuxtools-rst.readthedocs.org/zh_CN/latest/tool/crontab.html)