---
title: 使用mysqldump定时备份数据库的脚本
author: admin
type: post
date: 2007-11-23T14:45:41+00:00
url: /archives/208
IM_contentdowned:
 - 1
categories:
 - 数据库

---
每7天备份一次所有数据,每天备份binlog,也就是增量备份.

(如果数据少,每天备份一次完整数据即可,可能没必要做增量备份)

作者对shell脚本不太熟悉,所以很多地方写的很笨 🙂

开启 bin log

在[mysql][1] 4.1版本中,默认只有错误日志,没有其他日志.可以通过修改配置打开bin log.方法很多,其中一个是在/etc/my.cnf中的mysqld部分加入:

[mysqld]
log-bin

这个日志的主要作用是增量备份或者复制(可能还有其他用途).

如果想增量备份,必须打开这个日志.

对于数据库操作频繁的mysql,这个日志会变得很大,而且可能会有多个.

在数据库中flush-logs,或者使用mysqladmin,mysqldump调用flush-logs后并且使用参数delete-master-logs,这些日志文件会消失,并产生新的日志文件(开始是空的).

所以如果从来不备份,开启日志可能没有必要.

完整备份的同时可以调用flush-logs,增量备份之前flush-logs,以便备份最新的数据.

完整备份脚本

如果数据库数据比较多,我们一般是几天或者一周备份一次数据,以免影响应用运行,如果数据量比较小,那么一天备份一次也无所谓了.

下载假设我们的数据量比较大,备份脚本如下:(参考过网络上一个mysql备份脚本,致谢 :))

#!/bin/sh
\# mysql data backup scrīpt
\# by scud http://www.jscud.com
\# 2005-10-30
#
\# use mysqldump –help,get more detail.
#
BakDir=/backup/mysql
LogFile=/backup/mysql/mysqlbak.log
DATE=\`date +%Y%m%d\`
echo ” ” >> $LogFile
echo ” ” >> $LogFile
echo “——————————————-” >> $LogFile
echo $(date +”%y-%m-%d %H:%M:%S”) >> $LogFile
echo “————————–” >> $LogFile
cd $BakDir
DumpFile=$DATE.sql
GZDumpFile=$DATE.sql.tgz
mysqldump –quick –all-databases –flush-logs
–delete-master-logs –lock-all-tables > $DumpFile
echo “Dump Done” >> $LogFile
tar czvf $GZDumpFile $DumpFile >> $LogFile 2>&1
echo “[$GZDumpFile]Backup Success!” >> $LogFile
rm -f $DumpFile
#delete previous daily backup files:采用增量备份的文件,如果完整备份后,则删除增量备份的文件.
cd $BakDir/daily
rm -f *
cd $BakDir
echo “Backup Done!”
echo “please Check $BakDir Directory!”
echo “copy it to your local disk or ftp to somewhere !!!”
ls -al $BakDir

上面的脚本把mysql备份到本地的/backup/mysql目录,增量备份的文件放在/backup/mysql/daily目录下.

注意:上面的脚本并没有把备份后的文件传送到其他远程计算机,也没有删除几天前的备份文件:需要用户增加相关脚本,或者手动操作.

增量备份

增量备份的数据量比较小,但是要在完整备份的基础上操作,用户可以在时间和成本上权衡,选择最有利于自己的方式.

增量备份使用bin log,脚本如下:

#!/bin/sh
#
\# mysql binlog backup scrīpt
#
/usr/bin/mysqladmin flush-logs
DATADIR=/var/lib/mysql
BAKDIR=/backup/mysql/daily
###如果你做了特殊设置,请修改此处或者修改应用此变量的行:缺省取机器名,mysql缺省也是取机器名
HOSTNAME=\`uname -n\`
cd $DATADIR
FILELIST=\`cat $HOSTNAME-bin.index\`
##计算行数,也就是文件数
COUNTER=0
for file in $FILELIST
do
COUNTER=\`expr $COUNTER + 1 \`
done
NextNum=0
for file in $FILELIST
do
base=\`basename $file\`
NextNum=\`expr $NextNum + 1\`
if [ $NextNum -eq $COUNTER ]
then
echo “skip lastest”
else
dest=$BAKDIR/$base
if(test -e $dest)
then
echo “skip exist $base”
else
echo “copying $base”
cp $base $BAKDIR
fi
fi
done
echo “backup mysql binlog ok”

增量备份脚本是备份前flush-logs,mysql会自动把内存中的日志放到文件里,然后生成一个新的日志文件,所以我们只需要备份前面的几个即可,也就是不备份最后一个.

因为从上次备份到本次备份也可能会有多个日志文件生成,所以要检测文件,如果已经备份过,就不用备份了.

注:同样,用户也需要自己远程传送,不过不需要删除了,完整备份后程序会自动生成.

访问设置

脚本写完了,为了能让脚本运行,还需要设置对应的用户名和密码,mysqladmin和mysqldump都是需要用户名和密码的,当然可以写在脚本中,但是修改起来不太方便,假设我们用系统的root用户来运行此脚本,那么我们需要在/root(也就是root用户的home目录)创建一个.my.cnf 文件,内容如下

[mysqladmin]
password =password
user= root
[mysqldump]
user=root
password=password

注: 设置本文件只有root可读.(chmod 600 .my.cnf )

此文件说明程序使用mysql的root用户备份数据,密码是对应的设置.这样就不需要在脚本里写用户名和密码了.

自动运行

为了让备份程序自动运行,我们需要把它加入crontab.

有2种方法,一种是把脚本根据自己的选择放入到/etc/cron.daily,/etc/cron.weekly这么目录里.

一种是使用crontab -e放入到root用户的计划任务里,例如完整备份每周日凌晨3点运行,日常备份每周一-周六凌晨3点运行.

具体使用,请参考crontab的帮助.

 [1]: /?tag=mysql