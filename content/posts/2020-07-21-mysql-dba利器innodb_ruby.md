---
title: MySQL DBA利器innodb_ruby
author: admin
type: post
date: 2020-07-21T09:25:01+00:00
url: /archives/20090
categories:
 - MySQL

---
## innodb_ruby简介 

innodb_ruby是一款用ruby写的用来分析 innodb 物理文件的专业DBA工具，可以通过这款工具来窥探innodb内部的一些结构。
注意不要在生产环境中使用此工具，以避对线上服务造成影响。官方网址 [https://rubygems.org/gems/innodb_ruby](https://rubygems.org/gems/innodb_ruby)。

注意如果(Linux)平台安装中遇到错误一般情况是由于缺少依赖库造成的，可以先安装 sudo apt-get install libxslt1-dev libxml2-dev 相关库。

## 命令语法 

在执行以下命令时，建议切换到MySQL 的 datadir 目录里。

```
sxf@ubuntu:~$ innodb_space --help

Usage: innodb_space <options> <mode>
innodb_space <选项> <模式>
命令主要分 options 和 mode 两大部分。

Invocation examples:

  innodb_space -s ibdata1 [-T tname [-I iname]] [options] <mode>
    Use ibdata1 as the system tablespace and load the tname table (and the
    iname index for modes that require it) from data located in the system
    tablespace data dictionary. This will automatically generate a record
    describer for any indexes.

    参数：
    -s 参数指的是系统表空间文件 ibdata1, 这个一般在datadir目录里可以找到。
    -T 数据表名称，一般为数据库其中一个表的物理文件路径
    -I 表示索引的名称, 如果是主键的话，直接填写 -I PRIMARY 即可，此时可省略此参数

    如 innodb_space -s ibdata1 -T lab/tb space-indexes,则表示查看lab数据库的tb表的索引统计信息

  innodb_space -f tname.ibd [-r ./desc.rb -d DescClass] [options] <mode>
    Use the tname.ibd table (and the DescClass describer where required).

The following options are supported:

  --help, -?
    Print this usage text.

  --trace, -t
    Enable tracing of all data read. Specify twice to enable even more
    tracing (including reads during opening of the tablespace) which can
    be quite noisy.

  --system-space-file, -s <arg>
    Load the system tablespace file or files <arg>: Either a single file e.g.
    "ibdata1", a comma-delimited list of files e.g. "ibdata1,ibdata1", or a
    directory name. If a directory name is provided, it will be scanned for all
    files named "ibdata?" which will then be sorted alphabetically and used to
    load the system tablespace.

  --table-name, -T <name>
    Use the table name <name>.
    表名

  --index-name, -I <name>
    Use the index name <name>.
    索引名

  --space-file, -f <file>
    Load the tablespace file <file>.

  --page, -p <page>
    Operate on the page <page>.
    页数

  --level, -l <level>
    Operate on the level <level>.
    索引树层级数，一般不会超过3

  --list, -L <list>
    Operate on the list <list>.

  --fseg-id, -F <fseg_id>
      Operate on the file segment (fseg) <fseg_id>.

  --require, -r <file>
    Use Ruby's "require" to load the file <file>. This is useful for loading
    classes with record describers.

  --describer, -d <describer>
    Use the named record describer to parse records in index pages.

The following modes are supported:
模式项列表

  系统表空间
  system-spaces
    Print a summary of all spaces in the system.


  数据字典表(information_schema中数据库SYS_TABLES表内容，下同)
  data-dictionary-tables
    Print all records in the SYS_TABLES data dictionary table.

  data-dictionary-columns
    Print all records in the SYS_COLUMNS data dictionary table.

  data-dictionary-indexes
    Print all records in the SYS_INDEXES data dictionary table.

  data-dictionary-fields
    Print all records in the SYS_FIELDS data dictionary table.


  汇总表空间中的所有页信息，需要使用 --page/-p 参数指定页数
  space-summary
    Summarize all pages within a tablespace. A starting page number can be
    provided with the --page/-p argument.

  汇总表空间中的所有索引页信息，对于分析每个页记录填充率情况的时候很有用，同样需要使用--page/-p指定页数
  space-index-pages-summary
    Summarize all "INDEX" pages within a tablespace. This is useful to analyze
    page fill rates and record counts per page. In addition to "INDEX" pages,
    "ALLOCATED" pages are also printed and assumed to be completely empty.
    A starting page number can be provided with the --page/-p argument.

  与space-index-pages-summary差不多，但只显示一些摘要信息，需要配合参数一块使用
  space-index-fseg-pages-summary
    The same as space-index-pages-summary but only iterate one fseg, provided
    with the --fseg-id/-F argument.

  space-index-pages-free-plot
    Use Ruby's gnuplot module to produce a scatterplot of page free space for
    all "INDEX" and "ALLOCATED" pages in a tablespace. More aesthetically
    pleasing plots can be produced with space-index-pages-summary output,
    but this is a quick and easy way to produce a passable plot. A starting
    page number can be provided with the --page/-p argument.

  遍历空间中的所有页面，统计每个类型的页共占用了多少页
  space-page-type-regions
    Summarize all contiguous regions of the same page type. This is useful to
    provide an overall view of the space and allocations within it. A starting
    page number can be provided with the --page/-p argument.

  按类型汇总所有页面信息
  space-page-type-summary
    Summarize all pages by type. A starting page number can be provided with
    the --page/-p argument.

  表空间中所有索引统计信息（系统空间或每个文件表空间）
  space-indexes
    Summarize all indexes (actually each segment of the indexes) to show
    the number of pages used and allocated, and the segment fill factor.

  space-lists
    Print a summary of all lists in a space.

  space-list-iterate
    Iterate through the contents of a space list.

  space-extents
    Iterate through all extents, printing the extent descriptor bitmap.

  space-extents-illustrate
    Iterate through all extents, illustrating the extent usage using ANSI
    color and Unicode box drawing characters to show page usage throughout
    the space.

  space-extents-illustrate-svg
    Iterate through all extents, illustrating the extent usage in SVG format
    printed to stdout to show page usage throughout the space.

  space-lsn-age-illustrate
    Iterate through all pages, producing a heat map colored by the page LSN
    using ANSI color and Unicode box drawing characters, allowing the user to
    get an overview of page modification recency.

  space-lsn-age-illustrate-svg
    Iterate through all pages, producing a heat map colored by the page LSN
    producing SVG format output, allowing the user to get an overview of page
    modification recency.

  space-inodes-fseg-id
    Iterate through all inodes, printing only the FSEG ID.

  space-inodes-summary
    Iterate through all inodes, printing a short summary of each FSEG.

  space-inodes-detail
    Iterate through all inodes, printing a detailed report of each FSEG.

  通过递归整个B+树（通过递归扫描所有页面，而不仅仅是按列表的叶子页面）来执行索引扫描（执行完整索引扫描）
  index-recurse
    Recurse an index, starting at the root (which must be provided in the first
    --page/-p argument), printing the node pages, node pointers (links), leaf
    pages. A record describer must be provided with the --describer/-d argument
    to recurse indexes (in order to parse node pages).

  将索引作为索引递归进行递归处理，但在索引页中打印每条记录的偏移量
  index-record-offsets
    Recurse an index as index-recurse does, but print the offsets of each
    record within the page.

  index-digraph
    Recurse an index as index-recurse does, but print a dot-compatible digraph
    instead of a human-readable summary.

  打印指定 level 级别的所有page信息
  index-level-summary
    Print a summary of all pages at a given level (provided with the --level/-l
    argument) in an index.

  index-fseg-internal-lists
  index-fseg-leaf-lists
    Print a summary of all lists in an index file segment. Index root page must
    be provided with --page/-p.

  index-fseg-internal-list-iterate
  index-fseg-leaf-list-iterate
    Iterate the file segment list (whose name is provided in the first --list/-L
    argument) for internal or leaf pages for a given index (whose root page
    is provided in the first --page/-p argument). The lists used for each
    index are "full", "not_full", and "free".

  index-fseg-internal-frag-pages
  index-fseg-leaf-frag-pages
    Print a summary of all fragment pages in an index file segment. Index root
    page must be provided with --page/-p.

  page-dump
    Dump the contents of a page, using the Ruby pp ("pretty-print") module.

  page-account
    Account for a page's usage in FSEGs.

  page-validate
    Validate the contents of a page.

  页目录字典记录
  page-directory-summary
    Summarize the record contents of the page directory in a page. If a record
    describer is available, the key of each record will be printed.

  对一个页的所有记录进行汇总
  page-records
    Summarize all records within a page.

  详细说明一个页面的内容，并且根据类型进行着色显示
  page-illustrate
    Produce an illustration of the contents of a page.

  record-dump
    Dump a detailed description of a record and the data it contains. A record
    offset must be provided with -R/--record.

  record-history
    Summarize the history (undo logs) for a record. A record offset must be
    provided with -R/--record.

  undo-history-summary
    Summarize all records in the history list (undo logs).

  undo-record-dump
    Dump a detailed description of an undo record and the data it contains.
    A record offset must be provided with -R/--record.

```

## 参数详解 

测试数据库 lab ，表名 tb ，表结构如下，

```
CREATE TABLE `tb` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `num` int(11) NOT NULL,
  `age` tinyint(1) unsigned DEFAULT '13',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=40001 DEFAULT CHARSET=latin1;
```

这里先添加了3万多的测试数据。

**系统表空间 system-spaces**

```
root@ubuntu:/var/lib/mysql# innodb_space -s ibdata1 -T lab/tb system-spaces
name                            pages       indexes
(system)                        768         7
lab/tb                          448         1
mysql/engine_cost               6           1
mysql/gtid_executed             6           1
mysql/help_category             7           2
mysql/help_keyword              15          2
mysql/help_relation             9           1
mysql/help_topic                576         2
mysql/innodb_index_stats        6           1
mysql/innodb_table_stats        6           1
mysql/plugin                    6           1
mysql/server_cost               6           1
mysql/servers                   6           1
mysql/slave_master_info         6           1
mysql/slave_relay_log_info      6           1
mysql/slave_worker_info         6           1
mysql/time_zone                 6           1
mysql/time_zone_leap_second     6           1
mysql/time_zone_name            6           1
mysql/time_zone_transition      6           1
mysql/time_zone_transition_type 6           1
sys/sys_config                  6           1
```

innodb\_space列出所有物理对象的数量。这些文件一般在相应数据库中可以找到扩展名为.ibd 的文件,如 sys库的sys\_config.ibd文件

**索引结构、数据分配情 space-indexes**

```
root@ubuntu:/var/lib/mysql# innodb_space -s ibdata1 -T lab/tb space-indexes
id          name                            root        fseg        fseg_id     used        allocated   fill_factor
43          PRIMARY                         3           internal    1           1           1           100.00%
43          PRIMARY                         3           leaf        2           75          96          78.12%
```

列说明：
name：索引的名称，PRIMARY代表的就是聚集索引，因为InnoDB表是聚集所以组织表，行记录就是聚集索引；idx_c就是辅助索引的名称。
root：索引中根节点的page号；可以看出聚集索引的根节点是第3个page（为什么是从第三个page开始，看下文space-page-type-regions），辅助索引的根节点是第4个page。
fseg：page的说明，internal表示非叶子节点或属于根节点，leaf表示叶子节点（也就是数据页）。
used：索引使用了多少个page，可以看出聚集索引的根节点点使用了1个page，叶子节点使用了3个page；辅助索引idx_c的叶子节点使用了1个page。
allocated：索引分配了多少个page，可以看出聚集索引的根节点分配了1个page，叶子节点分配了3个page；辅助索引idx_c的叶子节点分配了1个page
fill_factor：索引的填充度，所有的填充度都是100%。

遍历空间中的所有页面，统计每个类型的页共占用了多少 space-page-type-regions

```
start       end         count       type
0           0           1           FSP_HDR
1           1           1           IBUF_BITMAP
2           2           1           INODE
3           5           3           INDEX
6           6           1           FREE (INDEX)
7           36          30          INDEX
37          63          27          FREE (ALLOCATED)
64          106         43          INDEX
107         127         21          FREE (ALLOCATED)
```

列说明：
start：从第几个page开始。
end：从第几个page结束。
count：占用了多少个page。
type：page的类型。

从上面的结果可以看出：“FSP\_HDR”、“IBUF\_BITMAP”、“INODE”是分别占用了0，1，2号的page，从3号page开始才是存放数据和索引的页（Index）
接下来，根据得到的聚集索引和辅助索引的根节点来获取索引上的其他page的信息。

**索引级数统计信息 index-level-summary**

```
root@ubuntu:/var/lib/mysql# innodb_space -s ibdata1 -T lab/tb -I PRIMARY -l 0 index-level-summary
page    index   level   data    free    records min_key
4       43      0       14952   1036    534     id=1
5       43      0       14952   1036    534     id=535
7       43      0       14952   1036    534     id=1069
8       43      0       14952   1036    534     id=1603
9       43      0       14952   1036    534     id=2137
10      43      0       14952   1036    534     id=2671
11      43      0       14952   1036    534     id=3205
12      43      0       14952   1036    534     id=3739
13      43      0       14952   1036    534     id=4273
14      43      0       14952   1036    534     id=4807
15      43      0       14952   1036    534     id=5341
16      43      0       14952   1036    534     id=5875
17      43      0       14952   1036    534     id=6409
18      43      0       14952   1036    534     id=6943
19      43      0       14952   1036    534     id=7477
20      43      0       14952   1036    534     id=8011
21      43      0       14952   1036    534     id=8545
22      43      0       14952   1036    534     id=9079
23      43      0       14952   1036    534     id=9613
24      43      0       14952   1036    534     id=10147
25      43      0       14952   1036    534     id=10681
26      43      0       14952   1036    534     id=11215
27      43      0       14952   1036    534     id=11749
28      43      0       14952   1036    534     id=12283
29      43      0       14952   1036    534     id=12817
30      43      0       14952   1036    534     id=13351
31      43      0       14952   1036    534     id=13885
32      43      0       14952   1036    534     id=14419
33      43      0       14952   1036    534     id=14953
34      43      0       14952   1036    534     id=15487
35      43      0       14952   1036    534     id=16021
36      43      0       14952   1036    534     id=16555
64      43      0       14952   1036    534     id=17089
65      43      0       14952   1036    534     id=17623
66      43      0       14952   1036    534     id=18157
67      43      0       14952   1036    534     id=18691
68      43      0       14952   1036    534     id=19225
69      43      0       14952   1036    534     id=19759
70      43      0       14952   1036    534     id=20293
71      43      0       14952   1036    534     id=20827
72      43      0       14952   1036    534     id=21361
73      43      0       14952   1036    534     id=21895
74      43      0       14952   1036    534     id=22429
75      43      0       14952   1036    534     id=22963
76      43      0       14952   1036    534     id=23497
77      43      0       14952   1036    534     id=24031
78      43      0       14952   1036    534     id=24565
79      43      0       14952   1036    534     id=25099
80      43      0       14952   1036    534     id=25633
81      43      0       14952   1036    534     id=26167
82      43      0       14952   1036    534     id=26701
83      43      0       14952   1036    534     id=27235
84      43      0       14952   1036    534     id=27769
85      43      0       14952   1036    534     id=28303
86      43      0       14952   1036    534     id=28837
87      43      0       14952   1036    534     id=29371
88      43      0       14952   1036    534     id=29905
89      43      0       14952   1036    534     id=30439
90      43      0       14952   1036    534     id=30973
91      43      0       14952   1036    534     id=31507
92      43      0       14952   1036    534     id=32041
93      43      0       14952   1036    534     id=32575
94      43      0       14952   1036    534     id=33109
95      43      0       14952   1036    534     id=33643
96      43      0       14952   1036    534     id=34177
97      43      0       14952   1036    534     id=34711
98      43      0       14952   1036    534     id=35245
99      43      0       14952   1036    534     id=35779
100     43      0       14952   1036    534     id=36313
101     43      0       14952   1036    534     id=36847
102     43      0       14952   1036    534     id=37381
103     43      0       14952   1036    534     id=37915
104     43      0       14952   1036    534     id=38449
105     43      0       14952   1036    534     id=38983
106     43      0       13552   2460    484     id=39517
root@ubuntu:/var/lib/mysql# innodb_space -s ibdata1 -T lab/tb -I PRIMARY -l 1 index-level-summary
page    index   level   data    free    records min_key
3       43      1       1050    15168   75      id=1
root@ubuntu:/var/lib/mysql# innodb_space -s ibdata1 -T lab/tb -I PRIMARY -l 2 index-level-summary
page    index   level   data    free    records min_key
```

这里我们分别查看了0、1和2级别的信息，但2级别是没有任何信息输出的，所以这里的索引树高度是2。

列说明：

page 页数，可以看到并不一定是连续的
index 待确认
level 级数
data 数据大小
free 空闲大小
records 记录个数
min_key 最小记录id，每个page都会有一个最小记录id,二分法查找记录时使用.

**查看汇总页记录 page-records**

```
root@ubuntu:/var/lib/mysql# innodb_space -s ibdata1 -T lab/tb -p 3 page-records
Record 126: (id=1) → #4
Record 140: (id=535) → #5
Record 154: (id=1069) → #7
Record 168: (id=1603) → #8
Record 182: (id=2137) → #9
Record 196: (id=2671) → #10
Record 210: (id=3205) → #11
Record 224: (id=3739) → #12
Record 238: (id=4273) → #13
Record 252: (id=4807) → #14
Record 266: (id=5341) → #15
Record 280: (id=5875) → #16
Record 294: (id=6409) → #17
Record 308: (id=6943) → #18
Record 322: (id=7477) → #19
Record 336: (id=8011) → #20
Record 350: (id=8545) → #21
Record 364: (id=9079) → #22
Record 378: (id=9613) → #23
Record 392: (id=10147) → #24
Record 406: (id=10681) → #25
Record 420: (id=11215) → #26
Record 434: (id=11749) → #27
Record 448: (id=12283) → #28
Record 462: (id=12817) → #29
Record 476: (id=13351) → #30
Record 490: (id=13885) → #31
Record 504: (id=14419) → #32
Record 518: (id=14953) → #33
Record 532: (id=15487) → #34
Record 546: (id=16021) → #35
Record 560: (id=16555) → #36
Record 574: (id=17089) → #64
Record 588: (id=17623) → #65
Record 602: (id=18157) → #66
Record 616: (id=18691) → #67
Record 630: (id=19225) → #68
Record 644: (id=19759) → #69
Record 658: (id=20293) → #70
Record 672: (id=20827) → #71
Record 686: (id=21361) → #72
Record 700: (id=21895) → #73
Record 714: (id=22429) → #74
Record 728: (id=22963) → #75
Record 742: (id=23497) → #76
Record 756: (id=24031) → #77
Record 770: (id=24565) → #78
Record 784: (id=25099) → #79
Record 798: (id=25633) → #80
Record 812: (id=26167) → #81
Record 826: (id=26701) → #82
Record 840: (id=27235) → #83
Record 854: (id=27769) → #84
Record 868: (id=28303) → #85
Record 882: (id=28837) → #86
Record 896: (id=29371) → #87
Record 910: (id=29905) → #88
Record 924: (id=30439) → #89
Record 938: (id=30973) → #90
Record 952: (id=31507) → #91
Record 966: (id=32041) → #92
Record 980: (id=32575) → #93
Record 994: (id=33109) → #94
Record 1008: (id=33643) → #95
Record 1022: (id=34177) → #96
Record 1036: (id=34711) → #97
Record 1050: (id=35245) → #98
Record 1064: (id=35779) → #99
Record 1078: (id=36313) → #100
Record 1092: (id=36847) → #101
Record 1106: (id=37381) → #102
Record 1120: (id=37915) → #103
Record 1134: (id=38449) → #104
Record 1148: (id=38983) → #105
Record 1162: (id=39517) → #106
```

每一行代表一个page记录，id=1表示这个表中的记录最小主键id=1， #4则表示在页号是4。

上面我们使用 index-level-summary 查看的level 1级别的索引page 3中共有75条记录，最小id为1，这里通过 page-records确认了这一点。

这里查看的是聚集索引（主键索引），如果是普通索引的话，会看到打印内容有一些不一样，类似于 RECORD: (age=21) → (id=100) 这种的，即指向了主键值。

现在我们在看一下page 4中的内容

```
root@ubuntu:/var/lib/mysql# innodb_space -s ibdata1 -T lab/tb -p 4 page-records | head
Record 126: (id=1) → (num=1, age=13)
Record 154: (id=2) → (num=2, age=13)
Record 182: (id=3) → (num=3, age=13)
Record 210: (id=4) → (num=4, age=13)
Record 238: (id=5) → (num=5, age=13)
```

我们发现输出的内容与page 3 的有些不一样，这里输出的是完整的详情记录，但page 3是一个一条记录与页的对应关系，我们一般称其为页目录。

## 推荐阅读 
* http://vlambda.com/wz_xeipHG6Q3r.html
* https://www.cnblogs.com/cnzeno/p/6322842.html
* https://blog.csdn.net/weixin_34368949/article/details/91381989
* https://www.jianshu.com/p/c51873ea129a