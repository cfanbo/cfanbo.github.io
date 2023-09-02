---
title: 用cacti监控centos下mysql,memcache等服务状态
author: admin
type: post
date: 2011-05-08T06:56:05+00:00
url: /archives/9428
IM_contentdowned:
 - 1
categories:
 - 系统架构

---
CACTI测试OK
安装环境：CENTOS5.4
提前需要安装的组件：
1. mysql
2。APACHE
3。PHP

步骤：
一。安装 net-snmp

> yum install net-snmp*

注意加个*，把所有的咚咚都装上，否则没有cacti需要的命令.

二。安装 php-snmp

> yum install php-snmp

三.安装rrdtool，根据自己系统的版本，选择不同的RPM包

> [wget][1] [http://www.express.org/~wrl/rrdtool/rrdtool-1.2.30-1.el5.wrl.x86_64.rpm][2]
>
> wget [http://www.express.org/~wrl/rrdtool/rrdtool-devel-1.2.30-1.el5.wrl.x86_64.rpm][3]
>
> wget [http://www.express.org/~wrl/rrdtool/rrdtool-perl-1.2.30-1.el5.wrl.x86_64.rpm][4]

三个包必须一块安装

> rpm -ivh \*-1.2.30\*

四。安装cacti

> wget
> rpm -ivh cacti-0.8.6h.fc4.i386.rpm

基本上，该装的都装了.

五。 mysqladmin –user=root -p123456 create cacti

六。mysql cacti < /var/www/html/cacti/cacti.sql

七。shell> mysql -u root -p123456
mysql> GRANT ALL ON cacti.* TO  IDENTIFIED BY ‘cacti’;
mysql> flush privileges;

八。
vi /var/www/html/cacti/include/config.php
$database_password = “cacti”;
就改这一行口令就可以了，别的都是默认

九。
crontab -e
插入
\*/5 \* \* \* * cactiuser php /var/www/html/cacti/poller.php > /dev/null 2>&1
保存退出。

现在 键入看看吧.
进入cacti的设置界面之后，如果找不到那几个snmp的命令，把路径改称/usr/bin即可.注意graph permission三个条件是相乘关系

备注：
如果没图象，是/var/www/html/cacti里的文件权限问题．
＃chmod 777 -R /var/www/html/cacti／即可．

默认的用户名密码admin／admin

 [1]: http://www.express.org/%7Ewrl/rrdtool/
 [2]: http://www.express.org/%7Ewrl/rrdtool/rrdtool-1.2.30-1.el5.wrl.x86_64.rpm
 [3]: http://www.express.org/%7Ewrl/rrdtool/rrdtool-devel-1.2.30-1.el5.wrl.x86_64.rpm
 [4]: http://www.express.org/%7Ewrl/rrdtool/rrdtool-perl-1.2.30-1.el5.wrl.x86_64.rpm