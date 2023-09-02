---
title: 安装awstats来分析apache的访问日志
author: admin
type: post
date: 2008-12-04T05:27:13+00:00
excerpt: |
 AWStats: Advanced Web Statistics


 AWStats是在Sourceforge上发展很快的一个基于Perl的WEB日志分析工具。相对于另外一个非常优秀的开放源代码的日志分析工具Webalizer，AWStats的优势在于：
url: /archives/675
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - awstats

---
**AWStats: Advanced Web Statistics**

[AWStats][1]是在[Sourceforge][2]上发展很快的一个基于Perl的WEB日志分析工具。相对于另外一个非常优秀的开放源代码的日志分析工具[Webalizer][3]，AWStats的优势在于：

 1. 界面友好：可以根据浏览器直接调用相应语言界面（有简体中文版）
 [![](http://blog.haohtml.com/wp-content/uploads/2008/12/16144229358.jpg)][4]
 2. 基于Perl：并且很好的解决了跨平台问题，系统本身可以运行在GNU/Linux上或Windows上（安装了[ActivePerl][5]后）；分析的日志直接支持Apache格式 (combined)和IIS格式(需要修改)。Webalizer虽然也有[Windows平台版][6]，但目前已经缺乏维护；


 AWStats完全可以实现用一套系统完成对自身站点不同WEB服务器：GNU/Linux/Apache和Windows/IIS服务器的统一统计。
 3. 效率比较高：AWStats输出统计项目比Webalizer丰富了很多，速度仍可以达到Webalizer的1/3左右，对于一个日访问量百万级的站点，这个速度都是足够的；
 4. 配置/定制方便：系统提供了足够灵活但缺省也很合理的配置规则，需要修改的缺省配置不超过3，4项就可以开始运行，而且修改和扩展的插件还是比较多的；
 5. AWStats的设计者是面向精确的”Human visits”设计的，因此很多搜索引擎的机器人访问都被过滤掉了，因此有可能比其他日志统计工具统计的数字要低，来自公司内部的访问也可以通过IP过滤设置过滤掉。
 6. 提供了很多扩展的参数统计功能：使用ExtraXXXX系列配置生成针对具体应用的参数分析会对产品分析非常有用。

更多与其他工具：Webalizer, analog的比较请参考：
[http://awstats.sourceforge.net/#COMPARISON

][7]

AWStats的运行模式是这样的：

 1. 分析日志：运行后将这样的日志统计结果归档到一个AWStats的数据库（纯文本）里；
 2. 然后是输出：分两种形式
 * 一种是通过cgi程序读取统计结果数据库输出；
 * 一种是运行后台脚本将输出导出成静态文件；

++++++ 以上内容摘自《 [CHEDONG](http://www.chedong.com/tech/awstats.html)》++++++

接下来是整个我的整个安装过程

环境：RHEL AS 4.0 + Apache 2.2.2

**1. 修改Apache的配置文件 httpd.conf**

默认配置CustomLog的日记格式是common，改为combined，后者是awstats推荐的方式可以用来分析客户端浏览器的类型以及访问来源等。例如：
CustomLog “| /apache/httpd/bin/rotatelogs /apache/httpd/logs/access_%Y%m%d.log 86400” combined

这个日志配置让apache每天生成一个新的日志文件，其中%Y%m%d是年月日。

**2. 安装awstats**

从  下载安装包awstats-6.5.zip 并解压到某个目录{awstats}

awstats的脚本和静态文件缺省都在{awstats}/wwwroot目录下：将cgi-bin目录下的文件都拷贝到apache的cgi-bin目录下：mv {awstats}/wwwroot/cgi-bin {apache}/cgi-bin/awstats。将另外三个目录icon、js、css目录拷贝到apache的文档根目录。

给awstats的脚本增加可执行权限

> chmod +x {apache}/cgi-bin/awstats/*.pl

建立awstats数据目录：

> {apache}/cgi-bin/awstats/data

**3. 配置awstats**

进入{apache}/cgi-bin/awstats目录，修改awstats.model.conf文件名为awstats.[sitename].conf，其中[sitename]为你的网站名缩写，例如dlog，并打开文件，修改配置如下：

> LogFile=”{apache}/logs/access_%YYYY%mm%dd.log”
> SiteDomain=”你的域名”
> DirData=”{apache}/cgi-bin/awstats/data/”

存盘退出.

**4. 使用awstats**

重起apache服务，并访问地址：]就可以看到本文前面的截图，并且数据都是0。

我们需要手工的更新awstats的数据，执行下面命令

> {apache}/cgi-bin/awstats/awstats.pl -update -config=[sitename]

再次查看页面，就可以看到访问的统计信息。你可以把这个更新命令用定时作业来执行。

> crontab -e:
> #update awstats
> 10 8 \* \* * (cd {apache}/cgi-bin/awstats/; ./awstats.pl －update -config=[sitename])

**5. awstats的安全性**

awstats本身并没有对访问进行任何限制，因此我们必须通过apache的机制来实现，在httpd.conf末尾增加配置如下：

 Order deny,allow

 AuthType Basic

 AuthName “Restricted Files”

 AuthUserFile conf/awstats_passwd

 require user awstats_admin

使用apache自带的工具htpasswd来生成一个用户名和口令

> {apache}/bin/htpasswd -c {apache}/conf/awstats_passwd awstats_admin

重起apache，这样以后每次访问awstats页面都要求输入正确的用户名和口令。

 [1]: http://awstats.sourceforge.net/
 [2]: http://sourceforge.net/
 [3]: http://www.webalizer.org/
 [4]: http://blog.haohtml.com/wp-content/uploads/2008/12/16144229358.jpg
 [5]: http://www.activestate.com/
 [6]: http://linux1.netconx.de/klaus/webalizer/
 [7]: http://awstats.sourceforge.net/#COMPARISON