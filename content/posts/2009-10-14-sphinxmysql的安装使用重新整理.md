---
title: sphinx+MySQL的安装使用(重新整理)
author: admin
type: post
date: 2009-10-14T01:25:33+00:00
excerpt: |
 一、MySQL+Sphinx+SphinxSE安装步骤：
 1、安装python支持（以下针对CentOS系统，其他Linux系统请使用相应的方法安装）
 yum install -y python python-devel

 2、编译安装LibMMSeg（LibMMSeg是为Sphinx全文搜索引擎设计的中文分词软件包，其在GPL协议下发行的中文分词法，采用Chih-Hao Tsai的MMSEG算法。LibMMSeg在本文中用来生成中文分词词库。）
url: /archives/2502
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Sphinx

---
一、MySQL+Sphinx+SphinxSE安装步骤：
1、安装python支持（以下针对CentOS系统，其他Linux系统请使用相应的方法安装）
yum install -y python python-devel

2、编译安装LibMMSeg（LibMMSeg是为Sphinx全文搜索引擎设计的中文分词软件包，其在GPL协议下发行的中文分词法，采用Chih-Hao Tsai的MMSEG算法。LibMMSeg在本文中用来生成中文分词词库。）

以下压缩包“sphinx-0.9.8-rc2-chinese.zip”中包含mmseg-0.7.3.tar.gz、sphinx-0.9.8-rc2.tar.gz以及中文分词补丁。

wget http://www.coreseek.com/uploads/sources/csft3_0b2.tar.gz
wget http://www.coreseek.com/uploads/sources/mmseg3_0b2.tar.gz
unzip sphinx-0.9.8-rc2-chinese.zip
tar zxvf mmseg3_0b2.tar.gz
cd mmseg3_0b2/
./configure
make
make install
cd ../

3、编译安装MySQL 5.1.26-rc、Sphinx、SphinxSE存储引擎
wget http://dev.mysql.com/get/Downloads/MySQL-5.1/mysql-5.1.26-rc.tar.gz/from/http://mirror.x10.com/mirror/mysql/
tar zxvf mysql-5.1.26-rc.tar.gz

tar zxvf csft3_0b2.tar.gz
cd csft3_0b2.tar.gz/
patch -p1 < ../sphinx-0.98rc2.zhcn-support.patch
patch -p1 < ../fix-crash-in-excerpts.patch
cp -rf mysqlse ../mysql-5.1.26-rc/storage/sphinx
cd ../

cd mysql-5.1.26-rc/
sh BUILD/autorun.sh
./configure –with-plugins=sphinx –prefix=/usr/local/mysql1/ –enable-assembler –with-extra-charsets=complex –enable-thread-safe-client –with-big-tables –with-readline –with-ssl –with-embedded-server –enable-local-infile
make && make install
cd ../
检查下是否安装好sphinx

show engines; 有个sphinx引擎cd csft3_0b2.tar.gz/

CPPFLAGS=-I/usr/include/python2.4

LDFLAGS=-lpython2.4

./configure –prefix=/usr/local/sphinx –with-mysql=/usr/local/mysql1

make

make install

cd ../cp /usr/local/sphinx/etc/sphinx.conf.dist /usr/local/sphinx/etc/sphinx.conf

4、创建Sphinx索引文件和MySQL数据文件存放目录


./usr/local/sphinx/bin/indexer test1 –config /usr/local/sphinx/etc/sphinx.conf


/usr/local/mysql1/bin/mysql_install_db –datadir=/usr/local/mysql1/var


5、创建MySQL配置文件

(1)、创建配置文件/mysql/3306/my.cnf


cd mysql-5.1.26-rc/

cp support-files/my-medium.cnf /mysql/3306/my.cnf

vim /mysql/3306/my.cnf

server_id=2(不同于主库和3406)

port=3306


(2)、创建配置文件/mysql/3406/my.cnf

cd mysql-5.1.26-rc/

cp support-files/my-medium.cnf /mysql/3306/my.cnf

vim /mysql/3306/my.cnf

server_id=3(不同于主库和3306)

port=3406

6、制作一份MySQL slave供搜索引擎使用

7、创建快捷启动、停止重启、杀死MySQL进程的脚本

cp support-files/mysqlserver /etc/rc.d/init.d/mysql

vim /etc/rc.d/init.d/mysql

conf=/mysql/3306/my.cnf

$bindir/mysqld_safe –defaults-file=/mysql/3306/my.cnf –datadir=$datadir –pid-file=$server_pid_file $other_args >/dev/null 2>&1 &

二、Sphinx配置


1、生成sphinx中文分词词库

(1)、词典的构造


mmseg -u unigram.txt


该命令执行后，将会产生一个名为unigram.txt.uni的文件，将该文件改名为uni.lib，完成词典的构造。需要注意的是，unigram.txt 必须为UTF-8编码。


(2)、词典文件格式

….

河 187

x:187

造假者 1

x:1

台北队 1

x:1

湖边 1

……


其中，每条记录分两行。其中，第一行为词项，其格式为：[词条]\t[词频率]。需要注意的是，对于单个字后面跟这个字作单字成词的频率，这个频率需要在 大量的预先切分好的语料库中进行统计，用户增加或删除词时，一般不需要修改这个数值；对于非单字词，词频率处必须为1。第二行为占位项，是由于 LibMMSeg库的代码是从Coreseek其他的分词算法库（N-gram模型）中改造而来的，在原来的应用中，第二行为该词在各种词性下的分布频 率。LibMMSeg的用户只需要简单的在第二行处填”x:1″即可。


用户可以通过修改词典文件增加自己的自定义词，以提高分词法在某一具体领域的切分精度，系统默认的词典文件在data/unigram.txt中。


(3)、Sphinx+MySQL搜索引擎的中文词库


2、创建Sphinx主索引文件、增量索引文件存放目录

mkdir /usr/local/sphinx/var/data/test1/

mkdir /usr/local/sphinx/var/data/test1stemmed/

3、创建Sphinx配置文件

#in MySQL

CREATE TABLE sphcounter

(

counterid INTEGER PRIMARY KEY NOT NULL,

max_doc_id INTEGER NOT NULL

);

#创建这张表用来标识上次重建主索引的id位置

# in sphinx.conf

source src1

{

type                    = mysql

sql_host                = localhost

sql_user                = root

sql_pass                = 123

sql_db                    = test

sql_port                = 3306    # optional, default is 3306

sql_sock                = /usr/local/mysql1/var/mysql.sock#以上都是用于连接数据库部分一看就懂

sql_query_pre            = SET NAMES utf8

sql_query_pre            =replace into sphcounter \

select 1,MAX(postid) from pa_gposts  #创建主索引前更改标识位置

sql_query                = \

SELECT postid, title,group_id \

FROM pa_gposts where postid <= \

(select max_doc_id from sphcounter where counterid=1)#主索引是id小于标识位置的部分

sql_attr_uint        = group_id#这个部分不被索引，但可以通过这个属性对结果进行排序

sql_ranged_throttle    = 0#每个查询之前先延迟0ms，也就是不延迟

#sql_query_info        = SELECT * FROM pa_gposts WHERE postid=$id

}

source src1throttled : src1

{

sql_query_pre=set names utf8

sql_query=SELECT postid, title \

FROM pa_gposts where postid >\

(select max_doc_id from sphcounter where counterid=1) #增量索引是id大于标识位置的部分

}

index test1

{

source            = src1 #数据源

path            = /usr/local/sphinx/var/data/test1/test1 #创建索引位置必须有目录/usr/local/sphinx/var/data/test1/

docinfo            = extern

mlock            = 0

min_word_len        = 1

charset_type        =  zh_cn.utf-8#支持中文索引必须为zh_cn.utf-8

charset_dictpath=/root/mmseg-0.7.3/data/ #词典的目录，词典下必须有uni.lib mmseg 生产的词典

min_prefix_len    = 0

min_infix_len        = 1

ngram_len                = 1

ngram_chars = U+4E00..U+9FBF, U+3400..U+4DBF, U+20000..U+2A6DF, U+F900..U+FAFF,\

U+2F800..U+2FA1F, U+2E80..U+2EFF, U+2F00..U+2FDF, U+3100..U+312F, U+31A0..U+31BF,\

U+3040..U+309F, U+30A0..U+30FF, U+31F0..U+31FF, U+AC00..U+D7AF, U+1100..U+11FF,\

U+3130..U+318F, U+A000..U+A48F, U+A490..U+A4CF

html_strip                = 0#不去除HTML标签

#其他的配置如min_word_len,charset_type,ngrams_chars,ngram_len这些则是支持中文检索需要设置的内容。


}

index test1stemmed : test1

{

source                  =src1throttled

path            = /usr/local/sphinx/var/data/test1stemmed/test1stemmed

}

indexer

{

mem_limit            = 256M

}

searchd

{

port                = 3312

log                    = /usr/local/sphinx/var/log/searchd.log

query_log            = /usr/local/sphinx/var/log/query.log

read_timeout        = 5

max_children        = 30

pid_file            = /usr/local/sphinx/var/log/searchd.pid

max_matches            = 1000

seamless_rotate        = 1

preopen_indexes        = 0

unlink_old            = 1

}


4、初始化sphinx中配置的全部索引

/usr/local/sphinx/bin/indexer –all –config /usr/local/sphinx/etc/sphinx.conf

5、创建2个shell脚本，一个用来创建主索引、一个用来创建增量索引（此步可以省略）


1.创建主索引脚本build_main_index.sh

#!/bin/sh

/usr/local/sphinx/bin/searchd –stop>>searchdlog

/usr/local/sphinx/bin/indexer test1 –config /usr/local/sphinx/etc/sphinx.conf>>mainindexlog

/usr/local/sphinx/bin/searchd>>searchdlog

赋予执行权限

chmod u+x build_main_index.sh

定时执行脚本

crontab -e

添加一行 ./root/build_delta_index.sh

2.创建增量索引脚本build_delta_index.sh

#!/bin/sh

/usr/local/sphinx/bin/searchd –stop >> searchdlog

/usr/local/sphinx/bin/indexer test1stemmed –config /usr/local/sphinx/etc/sphinx.conf >> deltaindexlog

/usr/local/sphinx/bin/indexer –merge test1 test1stemmed –config /usr/local/sphinx/etc/sphinx.conf >> deltaindexlog

/usr/local/sphinx/bin/searchd >> searchdlog


6、启动Sphinx守护进程

/usr/local/sphinx/bin/searchd –config /usr/local/sphinx/etc/sphinx.conf

关闭 /usr/local/sphinx/bin/searchd –config /usr/local/sphinx/etc/sphinx.conf –stop

7、配置服务器开机启动时需要自动执行的命令

8、创建Sphinx存储引擎表

CREATE TABLE `sphinx` (

`id` int(11) NOT NULL,

`weight` int(11) NOT NULL,

`query` varchar(255) NOT NULL,

`group_id` int(11) NOT NULL,

KEY `Query` (`Query`)

) ENGINE=SPHINX CONNECTION=’sphinx://localhost:3312/test1′;

与一般mysql表不同的是ENGINE=SPHINX CONNECTION=’sphinx://localhost:3312/test1′;，这里表示这个表采用SPHINXSE引擎，与sphinx的连接串是’sphinx://localhost:3312/test1，test1是索引名称

根据sphinx官方说明，这个表必须至少有三个字段，字段起什么名称无所谓，但类型的顺序必须是integer,integer,varchar，分别表示记录标识document ID,匹配权重weight与查询query，同时document ID与query必须建索引。另外这个表还可以建立几个字段，这几个字段的只能是integer或TIMESTAMP类型，字段是与sphinx的结果集绑定的，因此字段的名称必须与在sphinx.conf中定义的属性名称一致，否则取出来的将是Null值。


比如我在上面有定义了sql_attr_uint= group_id那么在这个表里头，你就可以再定义group_id字段。


三、如何通过SQL语句调用搜索引擎

1。简单的查询

select * from sphinx where query=’动画’;

2。联合查询：

select docs.title from test.pa_gposts docs join sphinx on (docs.postid=sphinx.id) where query=’制作;limit=1000′;


query=’关键字’ ，关键字就是你要搜索的关键字，如query=’CGArt’表示你要全文搜索CGArt


mode，搜索模式，值有：all,any,phrase,boolean,extended，默认是all

all, 匹配所有查询词（默认模式）

any, 匹配查询词中的任意一个

phrase, 将整个查询看作一个词组，要求按顺序完整匹配

boolean, 将查询看作一个布尔表达式 （参见 节 4.2, “布尔查询语法”)

extended, 将查询看作一个 Sphinx 内部查询语言的表达式（参见节 4.3, “扩展的查询语法”）


sort，排序模式，必须是relevance,attr_desc,attr_asc,time_segments,extended中的一种，在所有模式中除了relevance外，

属性名（或用extended排序）前面都需要一个冒号。

… where query=’test;sort=attr_asc:group_id’;按照group_id升序排序

… where query=’test;sort=extended:@weight desc,group_id asc’;

relevance 模式, 按相关度降序排列（最好的匹配排在最前面）

attr_desc 模式, 按属性降序排列 （属性值越大的越是排在前面）

attr_asc 模式, 按属性升序排列（属性值越小的越是排在前面）

time_segments 模式, 先按时间段（最近一小时/天/周/月）降序，再按相关度降序

extended 模式, 按一种类似 SQL 的方式将列组合起来，升序或降序排列。

RELEVANCE 忽略任何附加的参数，永远按相关度评分排序。所有其余的模式都要求额外的排序子句，子句的语法跟具体的模式有关。

ATTR_ASC,ATTR_DESC 以及 TIME_SEGMENTS 这三个模式仅要求一个属性名。

RELEVANCE 模式等价于在扩展模式中按”@weight DESC, @id ASC”排序，

ATTR_ASC 模式等价于”attribute ASC, @weight DESC, @id ASC”，而

ATTR_DESC 等价于”attribute DESC, @weight DESC, @id ASC”。

TIME_SEGMENTS 模式 在 TIME_SEGMENTS 模式中，属性值被分割成“时间段”，然后先按时间段排序，再按相关度排序。

EXTENDED 模式 在 EXTENDED 模式中，您可以指定一个类似 SQL 的排序表达式，但涉及的属性（包括内部属性）不能超过 5 个，例如：

@relevance DESC, group_id ASC, @id DESC

已知的内部属性：

@id (match ID)

@weight (match weight)

@rank (match weight)

@relevance (match weight)

@rank 和@relevance 只是@weight 的额外别名。


offset，结果记录集的起始位置，默认是0


limit，从结果记录集中取出的数量，默认是20条


index，要搜索的索引名称

… where query=’test;index=test1′;

… where query=’test;index=test1,test2,test3;’;


minid,maxid，匹配最小与最大文档ID

weights，以逗号分割的分配给sphinx全文检索字段的权重列表

… where query=’test;weights=1,2,3;’;

filter,!filter，以逗号分隔的属性名与一堆要匹配的值

#只包括1,5,19的组

… where query=’test;filter=group_id,1,5,19;’;

#不包括3,11的组

… where query=’test;!filter=group_id,3,11′;

range,!range，逗号分隔的属性名一最小与最大要匹配的值

#从3至7的组

… where query=’test;range=group_id,3,7;’;

#不包括从5至25的组

… where query=’test;!range=group_id,5,25;’;

maxmatches，每个查询最大匹配的值

… where query=’test;maxmatches=2000;’;

groupby，group by 方法与属性

… where query=’test;groupby=day:published_ts;’;

… where query=’test;groupby=attr:group_id;’;

groupsort，group by 的排序

… where query=’test;gropusort=’@count desc’;


select count(*) from pa_gposts docs join sphinx on (docs.postid=sphinx.id) where query=’动画;limit=1000′;


搜索标题包含动画

select count(*) from pa_gposts docs join sphinx on (docs.postid=sphinx.id) where query=’@title动画;limit=100000;mode=extended’;


四、添加分词的操作及效果

1.添加分词儿童动画片

select docs.title from pa_gposts docs join sphinx on (docs.postid=sphinx.id) where query=’儿童动画片;limit=100000′;

+————————————————————————————————–+

| title                                                                                            |

+————————————————————————————————–+

| 儿童动画片儿童影视/动画连续剧  迅雷下载集                                     |

| 发精彩儿童动画片10部，下载从速                                                     |

| 【儿童节专题】【17部经典动画片下载,附名单】                                  |

| [图]儿童安全教育动画片《平安》                      |

| 十五部国产儿童动画片下载                                                             |

| 推荐不用注册就能下载数千首儿童歌曲、动画片、游戏、故事等育儿资源 |

| 求儿童动画片                                                                               |

| 儿童歌曲、儿童故事、儿童动画片下载                                              |

| 儿童动画片–童话合集23部                                                               |

+————————————————————————————————–+

9 rows in set (0.00 sec)

没添加之前被分割成儿童/动画片


vim unigram.txt    添加下面2行（参见2.1.2词典的格式）

儿童动画片 1

x:1

（附）查看分词

mmseg -d  tobe_segment.txt

其中，命令使用‘-d’开关指定词库文件所在的位置，参数dict_dir为词库文件（uni.lib ）所在的目录；tobe_segment.txt 为待切分的文本文件，必须为UTF-8编码。如果一切正确，mmseg会将切分结果以及所花费的时间显示到标准输出上。

mmseg -d mmseg-0.7.3/data a

论坛/x 里/x 有/x 没有/x 迪/x 斯/x 尼/x 的/x 小公/x 主/x 动画片/x ，/x 睡/x 美人/x ，/x 阿/x 拉丁/x ，/x 灰姑娘/x

2。生成字典

mmseg -u unigram.txt uni.lib

3。重启服务器重建索引

mysql restart 因为mysql的告诉缓存所以要重启mysql


bin/searchd –stop


bin/indexer test1


bin/searchd

4。查看结果

mysql> select docs.title from pa_gposts docs join sphinx on (docs.postid=sphinx.id) where query=’儿童动画片;limit=100000′;

+————————————————————–+

| title                                                        |

+————————————————————–+

| 发精彩儿童动画片10部，下载从速                 |

| 十五部国产儿童动画片下载                         |

| 儿童动画片儿童影视/动画连续剧  迅雷下载集 |

| 求儿童动画片                                           |

| 儿童歌曲、儿童故事、儿童动画片下载          |

| 儿童动画片–童话合集23部                           |

+————————————————————–+

6 rows in set (0.06 sec)


添加之后只搜出儿童动画片


五、增量索引测试


1。原始数据

mysql> select docs.title from pa_gposts docs join sphinx on (docs.postid=sphinx.id) where query=’词典;limit=100000′;

+—————————————————————————–+

| title                                                                       |

+—————————————————————————–+

| 孕妇小词典                                                             |

| 征婚魔鬼词典                                                          |

| 和大家分享一个很棒的在线学习词典，对小孩很有帮助的 |

| [转贴]女人流行词典                                                  |

| 你不得不看的魔鬼词典                                              |

+—————————————————————————–+

5 rows in set (0.13 sec)

insert into pa_gposts (title) values(‘词典的构造’);

bin/searchd –stop

2。创建增量索引

bin/indexer test1stemmed –config /usr/local/sphinx/etc/sphinx.conf

3。合并索引

bin/indexer –merge test1 test1stemmed –config /usr/local/sphinx/etc/sphinx.conf

bin/searchd

4。查看结果

mysql> select docs.title from pa_gposts docs join sphinx on (docs.postid=sphinx.id) where query=’词典;limit=100000′;

+—————————————————————————–+

| title                                                                       |

+—————————————————————————–+

| 孕妇小词典                                                             |

| 征婚魔鬼词典                                                          |

| 和大家分享一个很棒的在线学习词典，对小孩很有帮助的 |

| [转贴]女人流行词典                                                  |

| 你不得不看的魔鬼词典                                              |

| 词典的构造                                                             |

+—————————————————————————–+

6 rows in set (0.08 sec)