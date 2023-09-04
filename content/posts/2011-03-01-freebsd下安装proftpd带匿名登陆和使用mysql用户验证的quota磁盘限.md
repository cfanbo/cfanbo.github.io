---
title: '[教程]FreeBSD下安装proftpd带匿名登陆和使用MySQL用户验证的Quota磁盘限额安装教程'
author: admin
type: post
date: 2011-03-01T07:16:20+00:00
url: /archives/7871
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - proftpd
 - QUOTA

---
配置文件是根据原来版本的基础上修改的，所以有些指令可能默认配置文件里没有了,但并不影响使用。

**一.安装MySQL**

安装教程请参考:

**二.安装proftpd**

> #cd /usr/ports/ftp/proftpd-mysql
> #make install clean

[![](https://blogstatic.haohtml.com//uploads/2023/09/proftpd-mysql-quota.png)][1]

安装的时候会要求选择proftpd要安装的模块，选择好mysql和quota，其他的根据情况选择.这里系统默认安装的是proftpd-1.3.3d.tar.bz2.

配置系统自启动proftpd服务,用vi编辑/etc/rc.conf配置文件，在末尾加入一行:

> proftpd_enable=”YES”

**三、创建ftp用户、用户组和目录设置**
1、创建proftpd服务运行的用户和用户组（用于虚拟主机网站ftp用户）

> #pw groupadd FTPGRP -g 2001
> #pw adduser FTPUSR -u 2001 -g 2001 -d /var/ftp/haohtml -s /sbin/nologin
> #mkdir /var/ftp/haohtml
> #chown -R FTPUSR:FTPGRP /var/ftp/haohtml

2、创建匿名登陆用户映射的系统用户和用户组(ftp匿名登录,使用独立的组和用户)

> #pw groupadd ftp -g 2002
> #pw adduser ftp -u 2002 -g 2002 -d /var/ftp/incoming -s /sbin/nologin
> #mkdir /var/ftp/incoming
> #chown -R ftp:ftp /var/ftp/incoming

**四、配置proftpd.conf**

完成上面的安装，proftpd其实已经可以运行了，但是我们还需要根据自己的安装要求进行必要的配置，默认的配置文件为/usr/local/etc/proftpd.conf，配置备份文件为proftpd.conf.sample.

1、编辑proftpd.conf，下面是Michael的文件，大家完全照抄就基本可以了

#基本配置

ServerName “haohtml’s Ftp Site”

ServerType standalone

DefaultServer on

#设置用户登陆时不显示ftp服务器版本信息

ServerIdent off


#设置ftp服务使用的端口，可以修改

Port 21


#是否支持ipv6,默认支持(这里我将其关闭了)

# UseIPv6  off


#设置ftp目录的权限

Umask 022


#设置系统各种超时时间和重试次数

MaxLoginAttempts 3

TimeoutLogin 120

TimeoutIdle 600

TimeoutNoTransfer 900

TimeoutStalled 3600


#允许最大的同时连接用户数

MaxClients 10


#允许每个用户主机最大并发连接数

MaxClientsPerHost 3


#将DenyAll前面的#去掉，否则上传的文件将无法使用chmod命令来修改文件的权限.

AllowOverwrite no

#  DenyAll

AllowStoreRestart on

UseReverseDNS off


#如果shell为空时允许用户登录

RequireValidShell off


#将用户限制在自己的主目录下 ，这个设置很重要

DefaultRoot ~

#设置系统最大的进程数，防止dos攻击，默认是30

MaxInstances 10


#设置系统用于运行proftpd服务的用户和用户组（需要后面自己创建）

User FTPUSR

Group FTPGRP


#设置对全局的文件可以进行改写


AllowOverwrite on


#设置系统运行日志和文件传输日志

SystemLog                       /var/log/proftpd.syslog

TransferLog                     /var/log/proftpd.transferlog


######################## 下面是匿名登陆部分的设置 #################

#匿名用户登陆后访问的目录为 ftp用户的主目录

User ftp

Group ftp

#设置anonymous用户为系统实际用户ftp的别名

UserAlias anonymous ftp

#设置匿名用户同时在线最大用户数

#此用户数受限于前面的全局同时在线用户数

MaxClients 5

#设置welcome.msg文件作为用户登陆的提示信息文件

#.message文件作为用户每次进入的目录提示信息

DisplayLogin welcome.msg

DisplayFirstChdir .message

#设置一些特殊的权限，比如写文件和登陆限制等 （可以不用）

DenyAll

#DenyAll

#Order deny,allow

#Deny from 61.101.201.0/32

#Allow from all

######################## 完成匿名登陆的设置 #################

######################## 下面是磁盘限额Quota设置 ############

#启用磁盘限额

QuotaDirectoryTally on


#SQLHomedirOnDemand on,这个已经由CreateHome代替

CreateHome  on


#磁盘限额单位 b”|”Kb”|”Mb”|”Gb”

QuotaDisplayUnits “Mb”


QuotaEngine on


#磁盘限额日志记录

QuotaLog “/var/log/proftpd.quotalog”


#打开磁盘限额信息 “quote SITE QUOTA”命令

QuotaShowQuotas on

####################### 完成磁盘配额设置 ####################


####################### 下面是MySQL数据库部分设置 ###########

#数据库联接的信息

#FTP是数据库名，localhost是MySQL主机名，3306是mysql服务的端口号

#proftpd是连接数据库的用户名，testpwd是密码（如果没有密码留空）

SQLConnectInfo FTP@localhost:3306 proftpd testpwd


#数据库认证的类型

SQLAuthTypes Backend Plaintext


#指定用来做用户认证的表的有关信息。(将在后面创建)

SQLUserInfo FTPUSERS userid passwd uid gid homedir shell

SQLGroupInfo FTPGRPS groupname gid members


#数据库认证

SQLAuthenticate users groups usersetfast groupsetfast


#proftpd进行的mysql调用语句，别修改任何地方

SQLNamedQuery get-quota-limit SELECT “name, quota_type, per_session, limit_type, bytes_in_avail, bytes_out_avail, bytes_xfer_avail,files_in_avail, files_out_avail, files_xfer_avail FROM quotalimits WHERE name = ‘%{0}’ AND quota_type = ‘%{1}'”


SQLNamedQuery get-quota-tally SELECT “name, quota_type, bytes_in_used, bytes_out_used, bytes_xfer_used, files_in_used, files_out_used, files_xfer_used FROM quotatallies WHERE name = ‘%{0}’ AND quota_type = ‘%{1}'”


SQLNamedQuery update-quota-tally UPDATE “bytes_in_used = bytes_in_used + %{0}, bytes_out_used = bytes_out_used + %{1}, bytes_xfer_used = bytes_xfer_used + %{2}, files_in_used = files_in_used + %{3}, files_out_used = files_out_used + %{4}, files_xfer_used = files_xfer_used + %{5} WHERE name = ‘%{6}’ AND quota_type = ‘%{7}'” quotatallies


SQLNamedQuery insert-quota-tally INSERT “%{0}, %{1}, %{2}, %{3}, %{4}, %{5}, %{6}, %{7}” quotatallies


QuotaLimitTable sql:/get-quota-limit

QuotaTallyTable sql:/get-quota-tally/update-quota-tally/insert-quota-tally


至此，完成了proftpd.conf配置文件的编写.对于如何启用磁盘配额，请参考文章下面[**附：七，启用系统QUOTA磁盘配额**][2]

对于proftpd的权限设置详细请参考：

**五、完成MySQL数据库表配置**

完成proftpd.conf配置文件后，需要进行数据库表的配置，包括创建表和插入数据
1、登陆mysql或者使用phpmyadmin工具创建数据库 FTP,如果设置了root密码，则先进到mysql>提示符下，用create database dbname命令创建FTP数据库．

> #mysql -u root -p
> Enter password:
> mysql>create database FTP;
> mysql>use FTP;
> mysql>source /var/ftp.sql; //这里直接导入上面的sql脚本

2、运行下面的sql语句创建表和插入必要数据

— phpMyAdmin SQL Dump

— version 2.6.4-pl2

— http://www.phpmyadmin.net

—

— 主机: localhost

— 生成日期: 2005 年 11 月 03 日 14:23

— 服务器版本: 4.1.14

— PHP 版本: 4.4.0

—

— 数据库: `FTP`

—

— ——————————————————–

—

— 表的结构 `FTPGRPS`

—

CREATE TABLE `FTPGRPS` (

`groupname` text NOT NULL,

`gid` smallint(6) NOT NULL default ‘0’,

`members` text NOT NULL

) ENGINE=MyISAM;


—

— 导出表中的数据 `FTPGRPS`

—

INSERT INTO `FTPGRPS` VALUES (‘FTPGRP’, 2001, ‘FTPUSR’);

INSERT INTO `FTPGRPS` VALUES (‘ftp’, 2002, ‘ftp’);


— ——————————————————–

—

— 表的结构 `FTPUSERS`

—

CREATE TABLE `FTPUSERS` (

`userid` text NOT NULL,

`passwd` text NOT NULL,

`uid` int(11) NOT NULL default ‘0’,

`gid` int(11) NOT NULL default ‘0’,

`homedir` text,

`shell` text

) ENGINE=MyISAM;


—

— 导出表中的数据 `FTPUSERS`

—

INSERT INTO `FTPUSERS` VALUES (‘haohtml’, ‘123456’, 2001, 2001, ‘/var/ftp/haohtml’, ”);

INSERT INTO `FTPUSERS` VALUES (‘ftp’, ‘123456’, 2002, 2002, ‘/var/ftp/incoming’, ”);


— ——————————————————–

—

— 表的结构 `quotalimits`

—

CREATE TABLE `quotalimits` (

`name` varchar(30) default NULL,

`quota_type` enum(‘user’,’group’,’class’,’all’) NOT NULL default ‘user’,

`per_session` enum(‘false’,’true’) NOT NULL default ‘false’,

`limit_type` enum(‘soft’,’hard’) NOT NULL default ‘soft’,

`bytes_in_avail` float NOT NULL default ‘0’,

`bytes_out_avail` float NOT NULL default ‘0’,

`bytes_xfer_avail` float NOT NULL default ‘0’,

`files_in_avail` int(10) unsigned NOT NULL default ‘0’,

`files_out_avail` int(10) unsigned NOT NULL default ‘0’,

`files_xfer_avail` int(10) unsigned NOT NULL default ‘0’

) ENGINE=MyISAM;


—

— 导出表中的数据 `quotalimits`

—

— 设置haohtml用户，磁盘配额1G，可以上传下载流量2G，最多文件数10个

INSERT INTO `quotalimits` VALUES (‘haohtml’, ‘user’, ‘false’, ‘soft’, 1.024e+09, 0, 2.048e+09, 10, 0, 0);


— ——————————————————–

—

— 表的结构 `quotatallies`

—

CREATE TABLE `quotatallies` (

`name` varchar(30) NOT NULL default ”,

`quota_type` enum(‘user’,’group’,’class’,’all’) NOT NULL default ‘user’,

`bytes_in_used` float NOT NULL default ‘0’,

`bytes_out_used` float NOT NULL default ‘0’,

`bytes_xfer_used` float NOT NULL default ‘0’,

`files_in_used` int(10) unsigned NOT NULL default ‘0’,

`files_out_used` int(10) unsigned NOT NULL default ‘0’,

`files_xfer_used` int(10) unsigned NOT NULL default ‘0’

) ENGINE=MyISAM;


如果你想设置quota，只要在**quotalimits**表里设置一下就行了，这个表里的各个参数分别代表：

**quotalimits表**
name： – 用户帐号
quota type： – user, group, class, all (we use user)
per_session： – true or false (we use true)
limit_type： – 硬限制 or 软限制 (我们一般用硬限制)
bytes_in_avail： – 允许上传的字节数
bytes_out_avail： – 允许下载的字节数
bytes_xfer_avail： – 允许传输的字节数（包括上传/下载）
files_in_avail： – 允许上传的文件数
files_out_avail： – 允许下载的文件数
files_xfer_avail： – 允许传输的文件数（包括上传/下载）

**
**

**六、运行测试配置系统服务**
1、运行proftpd服务,直接#proftpd也可以．

> #/usr/local/sbin/proftpd

2、测试
在任何一台可以访问服务器的机器上测试ftp连接，使用haohtml用户和testftp口令登陆，然后运行quote SITE QUOTA命令查看设置的磁盘限额信息,然后测试使用anonymous匿名用户登陆测试.

> \# ftp 192.168.0.111
> Connected to 192.168.0.111.
> 220 ProFTPD 1.3.3d Server (ProFTPD Default Installation) [192.168.0.111]
> Name (192.168.0.111:root): haohtml
> 331 Password required for haohtml
> Password:
> 230 User haohtml logged in
> Remote system type is UNIX.
> Using binary mode to transfer files.
> ftp> quote SITE QUOTA
> 200-The current quota for this session are [current/limit]:
> Name: haohtml
> Quota Type: User
> Per Session: False
> Limit Type: Soft
> Uploaded Mb: 0.00/976.56
> Downloaded Mb: unlimited
> Transferred Mb: 0.00/1953.12
> Uploaded files: 0/10
> Downloaded files: unlimited
> Transferred files: unlimited
> 200 Please contact root@www.haohtml.com if these entries are inaccurate
> ftp>

3、监控和调试proftpd服务

> #/usr/local/sbin/proftpd proftpd -n -d 5 -c /usr/local/etc/proftpd.conf

这样在测试和连接ftp的时候，可以在主机上看到所有的proftpd运行信息

4、日志监控
可以使用下面的命令查看系统日志、传送日志等

> #tail -f /var/log/proftpd.syslog
> #tail -f /var/log/proftpd.transferlog

到此为止，已经基本完成了proftpd的配置.

**注意：**ftp用户要对访问的目录有读取权限，如对于/var/ftp/haohtml目录，用户haohtml需要有读取权限才可以．这里添加了两个操作系统账号，所以需要手动创建两个目录并指定权限.默认情况下在mysql里添加ftp用户信息后，只要使用指定的用户名和密码登录ftp，系统会自动创建目录，并分配好相应的目录权限．
默认情况下FreeBSD不支持磁盘配置的，详细教程参考：** [FreeBSD中使用QUOTA(磁盘配额)来限制用户空间](http://bbs.chinaunix.net/viewthread.php?tid=133891 "FreeBSD中使用QUOTA(磁盘配额)来限制用户空间")**

**附：七，启用系统QUOTA磁盘配额**

开启QUOTA支持
首先需要修改内核加入对quota的支持

> machine i386
> cpu I686_CPU
> #ident GENERIC
> ident MYGENERIC
> maxusers 0
> options QUOTA

#就是这行了。
修改好后重新编译内核。
然后在/etc/rc.conf里加入：

> enable_quotas=”YES”
> check_quotas=”YES”

这样你的系统就起用QUOTA了，你应当通过编辑/etc/fstab的某个文件系统的属性，加入QUOTA的支持。
下面的fstab文件就设置了在/pub文件系统上起用用户配额和组配额

> \# See the fstab(5) manual page for important information on automatic mounts
> \# of network filesystems before modifying this file.
> \# Device                Mountpoint      FStype  Options         Dump    Pass#
> /dev/ad0s1b             none            swap    sw              0       0
> /dev/ad0s1a             /               ufs     rw              1       1
> /dev/ad0s1h             /pub            ufs     rw,userquota,groupquota        2  2
> /dev/ad0s1e             /tmp            ufs     rw              2       2
> /dev/ad0s1g             /usr            ufs     rw              2       2
> /dev/ad0s1f             /var            ufs     rw              2       2
> /dev/acd0c              /cdrom          cd9660  ro,noauto       0       0
> proc                    /proc           procfs  rw              0       0

设置完fstab文件后，执行下面的命令打开quota

> \# quotacheck -av
> \# repquota -a

基本上前期的工作都已经做完了。

**附件：**
1、proftpd.conf
[proftpd.conf](/wp-content/uploads/2011/03/proftpd.conf)

2、FTP.sql
[FTP.sql](http://blog.haohtml.com/wp-content/uploads/2011/03/ftp.sql)

**参考资料：**

[http://www.chinaunix.net/jh/15/3028.html](http://www.chinaunix.net/jh/15/3028.html)

[1]: http://blog.haohtml.com/wp-content/uploads/2011/03/proftpd-mysql-quota.png
[2]: #quota