---
title: 使用 awstats 分析 Nginx 的访问日志
author: admin
type: post
date: 2010-12-20T05:05:23+00:00
url: /archives/7068
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - awstats
 - nginx

---

auth_basic     “admin”; #用户名

auth_basic_user_file　/opt/ngx/conf/admin.pass; #密码包路径

这篇文章内容部分有问题的，大家用的时候注意一下．特别是nginx的虚拟主机采用apache密码认证那一块的

>

```
auth_basic     "admin"; #用户名
/opt/ngx/conf/admin.pass; #密码包路径
```

```
正解的格式应该为：
```

>

```
auth_basic     "admin"; #用户名
auth_basic_user_file　/opt/ngx/conf/admin.pass; #密码包路径
```

**主要目录有三个：****１./data/web** #虚拟主机根目录 **２./data/webroot/awstats** #开始统计分析Awstats 日志（分析前需要将运行日志切割脚本 logcron.sh）,

 分析脚本为：

> #/usr/local/awstats/wwwroot/cgi-bin/awstats.pl -update -config=www.moabc.net**３./data/admin_web/awstats** #根据分析的Awstats日志生成静态页面,以便可以通过浏览器直接访问,运行脚本为：

> # /usr/local/awstats/tools/awstats_buildstaticpages.pl -update -config=www.moabc.net -lang=cn -dir=/data/admin_web/awstats -awstatsprog=/usr/local/awstats/wwwroot/cgi-bin/awstats.pl

Nginx 产生日志 –> 日志切割 –> Nginx 继续产生日志 –> 另存切割日志 –> 交由Awstats统计 –> 生成结果配置的时候有一个icons图片的目录，用的时候也需要做个别名过去，这样显示的时候有些图片才可以正常显示出来的．

对于分析日志文件的路径信息和生成分析结果的存放路径需要修改配置文件里，实例中的为:

#vi /etc/awstats/awstats.www.moabc.net.conf

找到统计的日志文件的路径

LogFile=”/var/log/httpd/mylog.log”

改为

LogFile=”/opt/nginx/logs/access_%YYYY-0%MM-0%DD-0.log

对于分析结果没有指定，只要修改

DirDate = “/data/webroot/awstats”

变量就可以了．