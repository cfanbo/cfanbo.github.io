---
title: Linux(nginx)下安装awstats日志分析软件
author: admin
type: post
date: 2011-09-01T09:24:21+00:00
url: /archives/11129
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - awstats
 - Linux
 - nginx

---
这里用的是centos的系统，linux上的安装方法基本上都一样的．

**一．下载awstats软件**

从地址可以下载，这里使用的是最新的7.0的版本

> #cd /usr/local
> #wget [http://cdnetworks-kr-1.dl.sourceforge.net/project/awstats/AWStats/7.0/awstats-7.0.zip](http://cdnetworks-kr-1.dl.sourceforge.net/project/awstats/AWStats/7.0/awstats-7.0.zip)#unzip awstats-7.0.zip awstats
> #chmod +x /usr/local/awstats/tools/awstats_configure.pl
> #chmod +x /usr/local/awstats/wwwroot/cgi-bin/awstats.pl
> #chmod +x /usr/local/awstats/tools/awstats_buildstaticpages.pl
> #用来存放swstats的数据文件
> #mkdir /var/lib/awstats

**二．创建配置文件**

> #cd /usr/local/awstats/tools/
> #perl ./awstats_configure.pl

根据提示信息进行相应的操作即可．好像第一步让输入web的配置文件，由于这里用的是nginx．不是apache的．所以输入none跳过即可．

在后面提示的域名里我们输入我们要分析的域名，如 [www.haohtml.com](http://www.haohtml.com)

这里会自动创建/etc/awstats/awstats.www.haohtml.com.conf配置文件．里面有些日志文件的路径我们需要修改为实际日志的路径即可．还有日志的默认格式的，要写成

> LogFormat=”%host %other %logname %time1 %methodurl %code %bytesd %refererquot %uaquot”

这种格式才可以的．

我这里web的日志位置在/var/log/httpd/haohtml/access.log

**三．配置Nginx**

为了方便，我们直接以独立域名的形式访问，如 [http://awstats.haohtml.com](http://www.haohtml.com)

配置如下：

> server {
> server_name  awstats.haohtml.com;
> charset gb2312;
>
> ……
> root   /data/wwwroot/awstats;
>
> access_log /var/log/httpd/haohtml/access.log;
>
> #这里将日志关闭
> access_log off;
>
> }

重启Nginx，便虚拟主机配置生效．如果对虚拟主机配置不懂的话，可以参考：

**四．定时分割日志文件**

> #vi **/root/cut\_nginx\_log.sh
>** #!/bin/bash
> mv /var/log/httpd/haothml/access.log /var/log/httpd/haothml/access_$(date -d “today” +”%Y%m%d”).log
> kill -USR1 \`cat /usr/local/nginx/logs/nginx.pid\`

添加脚本执行权限

> #chmod +x /root/cut\_nginx\_log.sh

注意，执行完此命令后，会自动将access.log日志文件改名为当天日期为文件名的文件．一般在晚上24时前一两分钟执行此命令，放在crontab定时执行．

**五.创建awstats日志报告html目录**

> mkdir -p /data/wwwroot/awstats/
> cp -R /usr/local/awstats/wwwroot/css /data/wwwroot/awstats/
> cp -R /usr/local/awstats/wwwroot/icon /data/wwwroot/awstats/

** 六．手动生成awstats日志分析数据库信息**

> /usr/local/awstats/wwwroot/cgi-bin/awstats.pl -update -config=www.haohtml.com

执行完以后，会把从日志里分析出来的结果存放在awstats的数据库里(结果存放位置为/var/lib/awstats),现在还没有办法查看，需要生成html日志报告才可以查看.

**七．生成html报告**

上面我们已经分析过了日志信息，这里我们将分析结果转换成html的格式，以便以网页的形式进行访问

> /usr/local/awstats/tools/awstats_buildstaticpages.pl -update -config=www.haohtml.com -dir=/data/wwwroot/awstats -lang=cn -awstatsprog=/usr/local/awstats/wwwroot/cgi-bin/awstats.pl

这时，可以在浏览器里通过域名http://awstats.haohtml.com/awstats.www.haohtml.com.html访问了．

**八．放在crontab计划任务中**

在上面我们只是手动生成了html报告文件，这里让它自动完成这些操作.

编辑/etc/crontab文件，在最后添加以下两行

> #vi /etc/crontab
> 59 23 \* \* *  /root/cut\_nginx\_log.sh
> 58 23 \* \* * /usr/local/awstats/tools/awstats_buildstaticpages.pl -update -config=www.haohtml.com -dir=/data/wwwroot/awstats -lang=cn -awstatsprog=/usr/local/awstats/wwwroot/cgi-bin/awstats.pl >/dev/null 2>&1

 注意上面生成报道shell和切割nginx日志的时间先后顺序,先生成报道再切换日志的.

也可以让每10分钟生成一下静态报道页面.到23时58分时重新生成报道信息,在23时59分的时候进行切割日志.

\*/10 \* \* \* * /usr/local/awstats/tools/awstats_buildstaticpages.pl -update -config=www.haohtml.com -dir=/data/wwwroot/awstats -lang=cn -awstatsprog=/usr/local/awstats/wwwroot/cgi-bin/awstats.pl >/dev/null 2>&1

对于crontab的更多介绍可以参考: