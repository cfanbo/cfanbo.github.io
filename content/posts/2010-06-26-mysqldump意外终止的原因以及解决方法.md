---
title: mysqldump意外终止的原因以及解决方法
author: admin
type: post
date: 2010-06-26T17:00:10+00:00
url: /archives/4113
IM_contentdowned:
 - 1
categories:
 - MySQL

---
mysqldump是非常重要的MySQL备份工具。然而在长年累月的使用过程中，TAOBAO多次出现了因mysqldump意外终止而导致备份 失败的情况。
以下是我们经常遇到的问题：

1、Lost connection to MySQL server at ‘reading initial communication packet’：
这个主要是因为DNS不稳定导致的。如果做了网络隔离，MySQL处于一个相对安全的网络环境，那么开启skip-name-resolve选项将会最大 程度避免这个问题。

2、Lost connection to MySQL server at ‘reading authorization packet’：
从MySQL获取一个可用的连接是多次握手的结果。在多次握手的过程中，网络波动会导致握手失败。增加connect\_timeout可以解决这个问题； 然而增加connect\_timeout并不能防止网络故障的发生，反而会引起MySQL线程占用。最好的解决办法是让mysqldump重新发起连接请 求。

3、Lost connection to MySQL server during query：
这个问题具备随机性，而淘宝MySQL的应用场景决定了我们无法多次备份数据以便重现问题。
然而我们注意到这个问题一般会在两种情况下会发生。一种是mysqldump *\*\\*\* | gzip \*\*\*\*；另外一种是mysqldump \*\*** > /nfs-file
注意，不管是gzip还是nfs都有一种特点，那就是它们影响了mysqldump的速度。从这个角度思考，是不是mysqldump从MySQL接受数 据包的速度不够快导致Lost connection to MySQL server during query错误呢？

为了定位到问题，我搭建了一个测试环境：
test@192.168.0.1：3306
CREATE TABLE \`test\` (
\`id\` bigint(20) NOT NULL auto_increment,
\`b\` varchar(2000) default NULL,
\`c\` varchar(2000) default NULL,
\`d\` varchar(2000) default NULL,
\`e\` varchar(2000) default NULL,
PRIMARY KEY (\`id\`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

insert into test(b,c,d,e) values (lpad(‘a’,1900,’b’), lpad(‘a’,1900,’b’), lpad(‘a’,1900,’b’), lpad(‘a’,1900,’b’));
多次复制数据使测试环境达到一定数据量。

192.168.0.2：
编写一个c++程序
#include
#include

using namespace std;

int main()
{
MYSQL conn;
MYSQL_RES *result;
MYSQL_ROW row;
my_bool reconnect = 0;

mysql_init(&conn);
mysql\_options(&conn, MYSQL\_OPT_RECONNECT, &reconnect);

if(!mysql\_real\_connect(&conn, “192.168.0.1″, “test”, “test”, “test”, 3306, NULL, 0))
{
fprintf(stderr, “Failed to connect to database: %s\n”, mysql_error(&conn));
exit(0);
}
else
{
fprintf(stdout, “Success to connect\n”);
}

mysql_query(&conn, “show variables like ‘%timeout%’”);
result = mysql\_use\_result(&conn);
while(row=mysql\_fetch\_row(result))
{
fprintf(stdout, “%-10s: %s\n”, row[0], row[1]);
}
mysql\_free\_result(result);
fprintf(stderr, “\n”);

mysql\_query(&conn, “select SQL\_NO_CACHE * from test.test”);
result = mysql\_use\_result(&conn);
while((row=mysql\_fetch\_row(result))!=NULL)
{
fprintf(stderr, “Error %d: %s\n”, mysql\_errno(&conn), mysql\_error(&conn));
fprintf(stdout, “%s\n”, row[0]);
sleep(100);
}
fprintf(stderr, “Error %d: %s\n”, mysql\_errno(&conn), mysql\_error(&conn));
mysql\_free\_result(result);
mysql_close(&conn);
return 1;
}

在这段代码里，sleep函数用来模拟NFS的网络延迟和gzip的运算时间。执行一段时间之后，Lost connection to MySQL server during query出现了，程序意外终止。在数据处理足够快的情况下，又会是怎样的结果？

将sleep的时间改为1，重新编译后发现程序能够完整跑完。根据[《MySQL Timeout解析》][1]上对net\_write\_timeout的解释，我们可以发现，mysqldump处理数据过慢（NFS、gzip引起） 会导致MySQL主动断开连接，此时mysqldump就会报Lost connection to MySQL server during query错误。经过多次测试，确定这个错误是由于net\_write\_timeout设置过短引起。

 [1]: http://rdc.taobao.com/blog/dba/html/433_mysql_timeout_analyze.html