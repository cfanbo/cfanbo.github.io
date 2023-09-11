---
title: FreeBSD下用mrtg监控本机流量、内存、cpu使用率、整网流量
author: admin
type: post
date: 2010-06-07T05:39:37+00:00
url: /archives/3816
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - mrgt

---
经常看到说mrtg的，说论坛里面的资料不对，其实不是不对，是有些说的不详细而已，我刚开始作的时候也是费了不少时间的，整理一下，发到这里吧，希望能 为后来的兄弟们省下时间
前言：我实验的机器是FreeBSD4.10，其他版本的应该也一样，其他unix like系统估计也是可以的，因为我只用过FreeBSD，不敢 肯定。另外我这篇文章的前提是你的机器上已经安装了apache，并能正常使用，如果没有请参考网上其他文 章安装，本文就不再赘述。

一：先介绍如何用mrtg来监控本机的流量
1：安装 snmp

cd /usr/ports/net-mgmt/p5-SNMP(好像没有这个目录的,可以用路径:/usr/ports/net-mgmt/net-snmp/”这)
make install clean
当中会叫你填写你的email、操作系统等等，直接回车即可。

可以用以下命令启动snmp，/usr/local/etc/rc.d/snmpd.sh start

2：安装 mrtg

cd /usr/ports/net-mgmt/mrtg
make install clean

3：配置 index.cfg文件监控服务器流量

/usr/local/bin/cfgmaker –output=/usr/www/mrtg/index.cfg public@192.168.0.1
然后修改index.cfg文件,主要修改以下内容，以符合你的实际情况，此处的192.168.0.1是你要监控的网卡的ip地址。

WorkDir: /usr/www/mrtg

Options[_]: growright, bits

Language:GB2312
########################让他5分钟执行一次##############
RunAsDaemon: Yes
Refresh:300
######或者可以这样：#################################
crontab -e
\*/5 \* \* \* * /usr/local/bin/mrtg /usr/www/mrtg/index.cfg
建议使用后面的方法，因为前者用RunAsDaemon的方式并不能使MRTG开机自动运行
#########################################

下面接着：
/usr/local/bin/mrtg /usr/www/mrtg/index.cfg

这个需要运行3次，前两次都会报错，不用去理会他，第3次就应该没有错误了，不过，
若是有问题的话，就需要改index.cfg，再执行直到没有错误发生为止。

制作首页index.html：

/usr/local/bin/indexmaker –output=/usr/www/mrtg/index.html /usr/www/mrtg/index.cfg

这样以后就可以通过：http://\***\***/mrtg/index.html看你的代理的流量了，注意此处是以你的apache主目录设置为 /usr/www来说的，你可以根据实际情况修改。

二、监控RAM-SWAP使用情况
在/usr/www/traffic/ram下建立ram.cfg,内容为：

LoadMIBs: /usr/local/share/snmp/mibs/UCD-SNMP-MIB.txt
Target[ramswap]: memAvailReal.0&memAvailSwap.0:public@192.168.0.1
##192.168.0.1是本机的ip地址
Options[ramswap]: nopercent,growright,gauge,noinfo
Title[ramswap]: RAM & SWAP 使用状况
PageTop[ramswap]: RAM & SWAP 使用状况
MaxBytes[ramswap]: 1000000000
kMG[ramswap]: k,M,G,T,P,X
Ylegend[ramswap]: Octets
ShortLegend[ramswap]: octets
LegendI[ramswap]: RAM 可使用
LegendO[ramswap]: Swap 可使用
Legend1[ramswap]: RAM 可使用单位
Legend2[ramswap]: Swap 可使用单位
Language:Chinese
WorkDir:/usr/www/traffic/ram

说明:与一般MRTG流量设定档大同小异，唯一的差別是来源数值。
memTotalSwap 全部的swap空间
memAvailSwap 剩余(可使用)的swap
memTotalReal 全部的内存空间
memAvailReal 剩余(可使用)的内存

然后执行：
/usr/local/bin/mrtg /usr/www/traffic/ram/ram.cfg
将会在/usr/www/traffic/ram下生成ramswap.html等文件。

这样以后就可以通过：http://\***\***/traffic/ram/ramswap.html看你的机器的内存使用情况了。

下面再让它5分钟执行一次：
crontab -e
\*/5 \* \* \* * /usr/local/bin/mrtg /usr/www/traffic/ram/ram.cfg

三、再来监控cpu使用率
安装bsdsar这个程序来显示cpu的使用状态
cd /usr/ports/sysutils/bsdsar/
make install
注意存档（/var/log/bsdsar.dat）会一直的变大，所以采用这个bsdsar必须要适时的将档案移往他处。并更名以作为日后的查询之用， 我是把它删除的，看看当前的就好了J
crontab -e
0 0 \* \* * /bin/rm /var/log/bsdsar.dat

在/usr/www/traffice/cpu下建立cpu.cfg,内容为：

Target[CPU]: \`/usr/www/traffic/cpu/mrtg-cpu\`
MaxBytes[CPU]: 100
Title[CPU]: CPU-Loading MRTG
PageTop[CPU]: CPU-Loading MRTG
Options[CPU]: gauge,growright
YLegend[CPU]: CPU Loading (%)
ShortLegend[CPU]: ％
WorkDir:/usr/plog/traffic/cpu
LegendO[CPU]: CPU系统负载
LegendI[CPU]: CPU使用者负载
Language:Chinese

说明:Target 乃是资料的取得方式 如同MRTG测流量时的public@community.
MaxBytes:限制绘图的最大 Loading 值,CPU Loading 的最高值就是 100% .
Title: HTML 网页的title .
PageTop: 网页页面的\*标题\*.
Options: 采用标准格式,并且让MRTG由右往左绘图.
YLegend: 图表的Y轴名称.
ShotLegend: 定义最小的单位(%).
WorkDir: 工作区域 也就是显示图表的位置.
Language: 用简体中文
LegendO[CPU] &  LegendI[CPU]: 下方的说明

设定MRTG-CPU Loading 的数据取得档案执行档:
/usr/local/www/data/mrtg/cpu/mrtg-cpu
此档权限需为可执行若用root执行则为700，内容:
#!/usr/bin/perl
$cpu_orig=\`/usr/local/bin/bsdsar -u >; /usr/www/traffic/cpu/bsdsar.tmp\`;
$cpu_str=\`/usr/bin/tail -1 /usr/www/traffic/cpu/bsdsar.tmp\`;
$val=(split(‘     ‘,$cpu_str))[1];
$val2=(split(‘     ‘,$cpu_str))[2];
$val=int($val);
$val2=int($val2);

print “$val\n”;
print “$val2\n”;
print “0\n”;
print “0\n”;

修改权限：chmod 700 mrtg-cpu

/usr/local/bin/mrtg /usr/www/traffic/cpu/cpu.cfg就会生成cpu.html等文件了。

可以通过http://\***\***/traffice/cpu/cpu.html看到cpu的使用情况了。

我设定每10分钟run一次.
\*/10 \* \* \* * /usr/local/bin/mrtg /usr/www/traffic/cpu/cpu.cfg

由于我是每隔10分钟run一次,所以原先装上bsdsar以后系统预设每隔20分钟执行一次的bsdsar_gather也要修正.
修改 /etc/crontab
#20,40   8-18    \*       \*       *       root    /usr/local/bin/bsdsar_gather
#0       \*       \*       \*       \*       root    /usr/local/bin/bsdsar_gather
\*/10 \* \* \* * /usr/local/bin/bsdsar_gather

四、下面来介绍一下如何监控整网的流量
我们的核心交换是cisco6509，下面的交换机是cisco3524，我们没有其他交换机，所以下面我说的命令是针对cisco的，其他的可参考手册 自己作相应的修改。
在6509的二层上设置：
set snmp rmon enable
set snmp community read-only mrtg
在FreeBSD机器上：

/usr/local/bin/cfgmaker –output=/usr/www/mrtg/6509.cfg mrtg@10.0.0.1
然后修改6509.cfg文件,主要修改以下内容，以符合你的实际情况，此处的10.0.0.1是6509的ip地址。
ee /usr/www/mrtg/6509.cfg
WorkDir: /usr/www/mrtg
Options[_]: growright, bits
Language:Chinese

/usr/local/bin/mrtg /usr/www/mrtg/6509.cfg
这个需要运行3次，前两次都会报错，不用去理会他，第3次就应该没有错误了，不过，
若是有问题的话，就需要改6509.cfg，再执行直到没有错误发生为止。

制作首页index.html：
/usr/local/bin/indexmaker –output=/usr/www/mrtg/6509.html /usr/www/mrtg/6509.cfg

让它每隔5分钟运行一下：
crontab -e
\*/5 \* \* \* * /usr/local/bin/mrtg /usr/www/mrtg/6509.cfg

五、监控下面交换机的流量
方法类似6509，只是命令有点不同：
snmp-server community mrtg ro，其他的照抄改一下文件的名字就行了。

六、后记：
这几个可能是大家比较关心的使用了，其他的我也没试过，大家如果有其他的利用请后续上，或者通知我wuming122@eyou.com，多谢！

七、FAQ：

################Q1############################
Q1：我在英文下
Max In: 935.6 kb/s (0.9%)
Max Out: 5306.4 kb/s (5.3%)
可是在中文下只显示
最大 流入: 935.6 $1$2/秒 (0.9%)
最大 流出: 5306.4 $1$2/秒 (5.3%)
这是怎么回事？
A1:编辑/usr/local/lib/perl5/site\_perl/5.8.5/locales\_mrtg.pm

查找到sub gb2312的下面
原来是这样的：
代码:

‘([kMG]?)([bB])/s’ =>; ‘$1$2/秒’,
‘([kMG]?)([bB])/min’ =>; ‘$1$2/分’,
‘([kMG]?)([bB])/h’ =>; ‘$1$2/时’,

改成这样：
代码:

‘([kMG]?)([bB])/s’ =>; ‘$1$2/秒’,
‘([kMG]?)([bB])/min’ =>; ‘$1$2/分’,
‘([kMG]?)([bB])/h’ =>; ‘$1$2/时’,
A2:修改你的mrtg.cfg文件。把语言一项改成:Chinese
也就是:
Language:Chinese
##########################################

#######################Q2##################
Q2：我运行了/usr/local/bin/cfgmaker –output=/usr/mrtg/6509.cfg mrtg@10.0.0.1
以后生成了四十多个Traffic Analysis for 1 — 6509等等，我们的6509上是加了个48口的板子，
总共应该有五十多个，现在只有四十多个是不是因为我在执行这个命令的时候只有这四十多个机
器是开着的？以后他们再开的时候会不会被监控到？会自动给我生成Traffic Analysis for 1 — 6509
这样的表吗？
A:没有处于connected的端口是不能被cfgmaker抓取的。要想实现不开机流量为0，开机以后就开始监控流量，
需要自己手动更改cfg文件，把所有未使用端口的注释去掉，注意空格和空行，一定要和cfgmaker生成的一致，
否则是抓不到的，然后再执行/usr/local/bin/mrtg /usr/www/mrtg/6509.cfg即 可。#######################################################

####################Q3################################
Q3：在这个机器上我只监控全部交换机上的流量，并不监控本身的流量，因为它也在6509上接着，
是不是就不需要运行snmpd了？
A：是交换机上的SNMP模组在做服务，与你的主机没关系，你的主机自然也就不用运行 SNMPD了，
但在交换机中抓到的你的这台主机的流量是反的，（即流入和流出是相反的，这个是显然的了）
#####################################################

####################Q4################################
Q4：我make install的时候提示有错误Couldn’t fetch it – please try to retrieve this ，装不上怎么办？
A：那是ports安装的时候需要的文件未能下载到，请确定你的机器已经连到网上并能访问国际互联 网，我知道很多学校都限制了访问国际网络，可以通过代理下载到那些文件放到/usr /ports/distfiles/下面，然后重新执行make install即可

来源:http://bbs.chinaunix.net/viewthread.php?tid=488199