---
title: 利用autobench测试web服务器极限并发数
author: admin
type: post
date: 2011-12-07T09:08:29+00:00
url: /archives/12205
IM_data:
 - 'a:6:{s:56:"http://salogs.com/wp-content/uploads/2010/07/results.gif";s:67:"http://blog.haohtml.com/wp-content/uploads/2011/12/8371_results.gif";s:55:"http://salogs.com/wp-content/uploads/2010/07/static.png";s:66:"http://blog.haohtml.com/wp-content/uploads/2011/12/2bdf_static.png";s:60:"http://salogs.com/wp-content/uploads/2010/07/total_php-2.png";s:71:"http://blog.haohtml.com/wp-content/uploads/2011/12/2c44_total_php-2.png";s:52:"http://salogs.com/wp-content/uploads/2010/07/cpu.png";s:63:"http://blog.haohtml.com/wp-content/uploads/2011/12/97e7_cpu.png";s:55:"http://salogs.com/wp-content/uploads/2010/07/memory.png";s:66:"http://blog.haohtml.com/wp-content/uploads/2011/12/9892_memory.png";s:53:"http://salogs.com/wp-content/uploads/2010/07/load.png";s:64:"http://blog.haohtml.com/wp-content/uploads/2011/12/7316_load.png";}'
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - 并发测试

---
# 一、目的

利用autobench工具结合httperf命令对web服务器进行测试，得出该服务器可以承载的最大并发连接数与最佳并发数。

# 二、测试工具

## 工具介绍

### 1、Httperf

[httperf][1] 是一款高性能的HTTP测试工具，使用它我们可以准确定位服务器的并发连接能力。下面介绍一下它的主要特征

（1） 可以观察测试客户端（并非被测服务器）在发起压力测试时的负载情况。这样在测试高并发的情况下可以准确的分析问题。（被测服务器无法承载高并发还是测试客户端无法发起过多请求）
（2）支持HTTP/1.1和SSL
（3）可以生成可扩展的测试计划

**下载**：

**安装**：

> \# tar xvzf httperf-0.9.0.tar.gz
> \# cd httperf-0.9.0
> #./configure
> \# make && make install

更多的使用方法参见man page。

### 2、autobench

[autobench][2] 是一款基于httperf的Perl脚本。它会在一次测试中调用多次httperf来对web服务器进行测试，每次会按照给定的参数增加并发连接数，将httperf的测试结果保存为CSV格式的文件，该文件可以被Excel直接读取，方便生成测试报告。借助于autobench自带的bench2graph工具可以生成漂亮的测试结果对比图，如下：

[![results.gif](http://salogs.com/wp-content/uploads/2010/07/results.gif)](http://salogs.com/wp-content/uploads/2010/07/results.gif "results.gif")

**下载**：http://www.xenoclast.org/autobench/downloads/

**安装**：

> \# yum install gd gnuplot pcre pcre-devel texinfo -y
> \# tar zxvf autobench-2.1.2.tar.gz
> \# cd autobench-2.1.2
> \# make && make install
> \# sed -i ‘s/postscript color/png xffffff/g’ /usr/local/bin/bench2graph （修改bench2graph脚本，否则生成的图像背景有问题）

使用方法：参见下文在实际测试中的使用

# 三、测试环境

**系统环境**
CentOS 5.3 64bit

**web****软件环境**
httpd-2.0.6
php5.2.6+ eAccelerator
php-fpm  开启20个php-cgi进程
nginx-0.7.67

在测服务器并发能力时会将apache与nginx对比测试

**硬件环境**

CPU:：E5504  2.00GHz
内存：1G
虚拟机环境

# 四、测试方法

1、  分别测试静态文件和动态php文件
2、  静态并发数从50开始，1500结束，增长幅度为50，动态5~100，增幅为5
3、  分别测试apache和nginx的并发能力，二者进行对比
4、  每次测试进行3次，最终结果求三次平均值
5、  每进行一次测试后均重启httpd或nginx（php-fpm）服务，释放内存后再进行下一轮测试
6、  为了减少磁盘IO，均关掉了访问日志

## 1、开始测试

### （1）静态文件

**测试命令**

> \# autobench –single\_host –host1=192.168.8.8 –port1=80 –uri1=/logo.gif  –quiet  –low\_rate=50 –high\_rate=1500 –rate\_step=50 –num\_call=1 –num\_conn=2000 –timeout=10 –file /tmp/result.tsv

**测试结果对比分析**
[![static.png](http://salogs.com/wp-content/uploads/2010/07/static.png)](http://salogs.com/wp-content/uploads/2010/07/static.png "static.png")

**测试结果总结：**

Apache与Nginx在并发50~1500时表现得都还可以，只不过在并发数达到1500后Apache的响应时间变得很长，由于系统环境的制约，我没有再测试大于1500的并发连接情况，但可以对比看出nginx在1500个并发连接的情况下还能保持较低的响应时间。

### （2）动态文件

**测试命令**

> \# autobench –single\_host –host1=192.168.8.8 –port1=80 –uri1=/test.php –quiet –low\_rate=5 –high\_rate=100 –rate\_step=5 –num\_call=1 –num\_conn=200 –timeout=10 –file /tmp/nginx_php1.tsv

**测试结果数据**

**并发连接数****nginx** **实际并发数****apache** **实际并发数****nginx** **应答时间****apache** **应答时间**
 5.0

 5.0

 5.0

 36.0

 36.3

 10.0

 10.0

 10.0

 33.0

 31.7

 15.0

 15.0

 15.0

 35.6

 31.9

 20.0

 19.9

 19.9

 36.8

 32.3

 25.0

 25.0

 25.0

 42.6

 36.2

 30.0

 29.8

 29.7

 45.1

 67.6

 35.0
 **34.3****32.4**
 115.5

 248.5

 40.0
 **35.5****34.9**
 396.8

 427.3

 45.0
 **35.3****33.4**
 699.0

 865.8

 50.0
 **35.7****30.5**
 962.3

 1394.0

 55.0
 **35.5****33.3**
 1103.9

 1354.7

 60.0
 **35.7****32.9**
 1269.2

 1471.3

 65.0
 **34.9****33.5**
 1445.8

 1573.6

 70.0
 **37.6****29.7**
 1458.0

 2049.5

 75.0
 **37.2****35.9**
 1610.1

 1496.8

 80.0
 **23.9****31.2**
 1588.2

 1993.3

 85.0
 **24.7****33.2**
 1674.9

 1880.2

 90.0
 **37.1****34.5**
 1838.6

 1946.0

 95.0
 **35.0****30.3**
 2027.4

 2387.2

 100.0
 **35.3****36.4**
 1996.3

 1904.5


测试数据

**测试结果对比图**
[![total_php-2.png](http://salogs.com/wp-content/uploads/2010/07/total_php-2.png)](http://salogs.com/wp-content/uploads/2010/07/total_php-2.png "total_php-2.png")

**测试总结**

由上面的报表以及这张曲线图可以看出，无论是apache还是nginx其php并发大于30其响应时间的就会直线上升，nginx略好于apache，因此可以判断这台服务器的php并发极限在30左右，当然这里指的php连接中是没有连接数据库的，也没有加入memcached等缓存机制。

### （3）高并发下系统资源情况

**CPU****使用情况对比
** [![cpu.png](http://salogs.com/wp-content/uploads/2010/07/cpu.png)](http://salogs.com/wp-content/uploads/2010/07/cpu.png "cpu.png")

**
内存使用情况对比**
[![memory.png](http://salogs.com/wp-content/uploads/2010/07/memory.png)](http://salogs.com/wp-content/uploads/2010/07/memory.png "memory.png")



**系统负载对比**
[![load.png](http://salogs.com/wp-content/uploads/2010/07/load.png)](http://salogs.com/wp-content/uploads/2010/07/load.png "load.png")

**分析测试结果**

CPU使用情况
在极限并发的情况下apache和nginx都占用很多CPU资源，这也是情理之中的事          情。nginx略好于apache平均占用90%而apache则在95%左右

内存使用情况
在内存对比中可以清楚的看到，nginx在极限并发的情况下内存控制得很好，到达一定程度后就不在变化了，而apache则会直线上升

负载情况
负载情况与内存情况类似，如果高并发时间很长的话apache服务器绝对会挂掉！nginx的负载也很高，但一直保持在10以下，这也和内存占用有关。

# 五、结论

利用httperf结合autobench可以很方便的测试出单台服务器的极限并发数，这样对服务器性能评估有很大帮助，借助于autobench的bench2graph脚本可以生成更为直观的对比图。

针对被测服务器，经过apache与nginx的对比发现，在静态文件的处理方面如果并发小于1500，apache和nginx之间的差距还是很小的。在动态php文件的并发测试中，nginx体现出其强大的性能优势，如果内存足够大，通过调整php-cgi数量，相信可以承载更多的并发连接。

# 六、附录

## 1、问题解决

**（1）当运行时报如下错误**

httperf: warning: open file limit > FD\_SETSIZE; limiting max. # of open files to FD\_SETSIZE

**解决方法**

其意思是说在httperf在发起连接请求时，单个进程已经无法再打开更多的文件描述符。在发起连接请求时httperf使用select()方法使用一个新的文件描述符。因此需要增加文件描述符限制

步骤1：编辑/etc/security/limits.conf 在最后添加下面两行内容
*       hard    nofile          102400
*       soft    nofile          102400

步骤2：编辑 /usr/include/bits/typesizes.h 文件修改_\_FD\_SET_SIZE常量值，如下
#define _\_FD\_SETSIZE            1024
修改为
#define _\_FD\_SETSIZE            102400

步骤3：重新编译httperf

## 2、参考文章

[http://www.cppblog.com/qiujian5628/archive/2008/03/10/44060.html
][3] [http://wiki.nginx.org/
][4]

**原创文章，转载请注明: 转自 [http://salogs.com][5]**

 [1]: http://code.google.com/p/httperf/
 [2]: http://www.xenoclast.org/autobench/
 [3]: http://www.cppblog.com/qiujian5628/archive/2008/03/10/44060.html
 [4]: http://wiki.nginx.org/
 [5]: http://salogs.com/2010/07/