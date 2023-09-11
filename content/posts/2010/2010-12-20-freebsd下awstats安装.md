---
title: '[教程]FreeBSD+nginx下Awstats安装(原创)'
author: admin
type: post
date: 2010-12-20T07:44:37+00:00
url: /archives/7071
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - awstats

---
**一.安装**

注：我安装的时候为7.0的,这篇文章本人没有进行测试

> #cd /usr/ports/www/awstats #make install

跟 Apache HTTP Server（以下称 Apache）不同的是，Apache 可以将日志输出通过管道的方式进行重新定向，依此来进行自动的日志切割。Nginx 在现今版本上还没能跟 Apache 一样，通过%YY等参数按日期分批创建日志，但是通过给 nginx 进程发送一个特定的信号，可以使 nginx 重新生成日志文件。我们可以定期执行一个 Shell 脚本来切换日志，重新命名或转移，具体的脚本如下：

>

```
# mv  /opt/nginx/logs/access.log /opt/nginx/logs/access_`date +%Y%m%d`.log
# killall –s USR1 nginx	#使用USR1参数通知Nginx进程切换日志文件
```

将以上脚本内容保存为文件名为 logcron.sh 存到自定的目录中，例如 /usr/local/etc/nginx/logcron.sh 使用 Crontab 让该脚本程序在每天晚上 11 点 59 分自动执行，即可做到按天创建日志。


安装之前，必须确认你的服务器上 Perl 的环境已经就绪。 查看当前环境 Perl 版本的命令是 perl –version 我们还需要对 Nginx 的日志格式做个小修改，不然 awstats 将无法进行统计。 例子如下(加粗部分)：

>

```
# vi /usr/local/etc/nginx/vhosts/www.mytest.com.conf

server {
listen       80;
server_name  www.mytest.com;

location ~ ^/web/ {
root   /data/www.mytest.com;
index  index.html;
error_log off;
charset gb2312;
}

log_format  new_log	#格式代称 (注意，如果有多个虚拟主机，代称不能一样)
'$remote_addr - $remote_user [$time_local] $request '
        '"$status" $body_bytes_sent "$http_referer" '
        '"$http_user_agent" "$http_x_forwarded_for"';
        access_log  logs/access.log new_log;	#日志生成路径

}
```

**二.配置**

安装后，会产生一个 /usr/local/www/awstats目录 。然后执行 tools 目录中的 awstats_configure.pl 配置向导，创建一个新的统计。

配置过程中会有两次提示错误的，注意一下。

>

```
www# cd /usr/local/www/awstats/tools
www# ./awstats_configure.pl
```

>
> —– AWStats awstats_configure 1.0 (build 1.9) (c) Laurent Destailleur —–
>
> This tool will help you to configure AWStats to analyze statistics for
>
> one web server. You can try to use it to let it do all that is possible
>
> in AWStats setup, however following the step by step manual setup
>
> documentation (docs/index.html) is often a better idea. Above all if:
>
> – You are not an administrator user,
>
> – You want to analyze downloaded log files without web server,
>
> – You want to analyze mail or ftp log files instead of web log files,
>
> – You need to analyze load balanced servers log files,
>
> – You want to ‘understand’ all possible ways to use AWStats…
>
> Read the AWStats documentation (docs/index.html).
>
> —–> Running OS detected: Linux, BSD or Unix
>
> Warning: AWStats standard directory on Linux OS is ‘/usr/local/awstats’.
>
> If you want to use standard directory, you should first move all content
>
> of AWStats distribution from current directory:
>
> /usr/local/www/awstats
>
> to standard directory:
>
> /usr/local/awstats
>
> And then, run configure.pl from this location.
>
> Do you want to continue setup from this NON standard directory [yN] ?N

#这里有一些提示信息，意思是说如果awstats标准目录为/usr/local/awstats.如果你的目录不是这个的话，需要手动把相关文件转移到/usr/local/awstats这个目录里.

这里我们安装的路径不是标准awstats路径的，所以需要手动把路径改过来。先输入 **N** ,下面把相应的文件转移到/usr/local/awstats这个目录里

> # cd /usr/local
> \# mv /usr/local/www/awstats /usr/local

下面我们重新运行一下配置命令 awstats_configure.pl.

> #/usr/local/awstats/tools/awstats_configure.pl

系统还会提示上面的信息，现在这里直接输入 y 即可.

紧接着就出现下面的提示信息的

>

```
-----> Check for web server install

Enter full config file path of your Web server.
Example: /etc/httpd/httpd.conf
Example: /usr/local/apache2/conf/httpd.conf
Example: c:\Program files\apache group\apache\conf\httpd.conf
Config file path ('none' to skip web server setup):
#> none  #因为我们这里用的是 Nginx，所以写 none，跳过。
```

回车

>

```
Your web server config file(s) could not be found.
You will need to setup your web server manually to declare AWStats
script as a CGI, if you want to build reports dynamically.
See AWStats setup documentation (file docs/index.html)

-----> Update model config file '/usr/local/awstats/wwwroot/cgi-bin/awstats.model.conf'
  File awstats.model.conf updated.

-----> Need to create a new config file ?
Do you want me to build a new AWStats config/profile
file (required if first install) [y/N] ?
#> y	#y 创建一个新的统计配置
```

输入y,回车

>

```
-----> Define config file name to create
What is the name of your web site or profile analysis ?
Example: www.mysite.com
Example: demo
Your web site, virtual server or profile name:
#> www.mytest.com		#统计网站的域名 例：www.mytest.com
```

回车

>

```
-----> Define config file path
In which directory do you plan to store your config file(s) ?
Default: /etc/awstats
Directory path to store config file(s) (Enter for default):
#>
```

使用默认直接回车，接下来便会出现以下的提示

```
又出现了下面的错误信息了，其实这里还是刚才的移到awstats目录的位置引起的。
```

>

```
-----> Create config file '/etc/awstats/awstats.www.mytest.com.conf'
Error: Failed to open '/usr/local/wwwroot/cgi-bin/awstats.model.conf' for read.
```

```
下面是解决办法：
```

>

```
#mkdir /usr/local/wwwroot
#mv /usr/local/awstats/cgi-bin /usr/local/wwwroot
```

```
好了，现在我们再重头来一次awstats_configure.pl吧，这里主要是想让大家知道安装过程中可能会出现哪些问题才这样写的,不要怕麻烦，对学习很有帮助的，呵呵。
```

>

```
#/usr/local/awstats/tools/awstats_configure.pl
```

```
中间的过程上面已经写过了，呵
```

当看到下面的提示信息基本上已经差不多快可以了，这里提示我们是否需要创建一个新的配置文件，对于第一次安装需要这个配置文件的。这里我们输入y。

> —–> Update model config file ‘/usr/local/wwwroot/cgi-bin/awstats.model.conf’
>
> File awstats.model.conf updated.
>
> —–> Need to create a new config file ?
>
> Do you want me to build a new AWStats config/profile
>
> file (required if first install) [y/N] ?y

输入y，回车

> —–> Create config file ‘/etc/awstats/awstats.www.mytest.com.conf’
>
> Config file /etc/awstats/awstats.www.mytest.com.conf created.
>
> —–> Add update process inside a scheduler
>
> Sorry, configure.pl does not support automatic add to cron yet.
>
> You can do it manually by adding the following command to your cron:
>
> /usr/local/wwwroot/cgi-bin/awstats.pl -update -config=www.mytest.com
>
> #回头把该命令填入crontab 按指定时间执行
> Or if you have several config files and prefer having only one command:
>
> /usr/local/tools/awstats_updateall.pl now
>
> Press ENTER to continue…

回车继续

> A SIMPLE config file has been created: /etc/awstats/awstats.www.mytest.com.conf #新配置文件所在的路径
>
> You should have a look inside to check and change manually main parameters.
> You can then manually update your statistics for ‘www.mytest.com’ with command:
> > perl awstats.pl -update -config=www.mytest.com
> You can also build static report pages for ‘www.mytest.com’ with command:
> > perl awstats.pl -output=pagetype -config=www.mytest.com
>
> Press ENTER to finish…

回车完成向导，接下来修改 www.mytest.com 的统计配置

> #vi /etc/awstats/awstats.www.mytest.com.conf

找到统计的日志文件的路径

> LogFile=”/var/log/httpd/mylog.log”

改为

> LogFile=”/usr/local/etc/nginx/logs/access_%YYYY-0%MM-0%DD-0.log #这里为nginx的路径信息，在上面的虚拟主机的地址写着的，上面用的是相对路径的，这时要用绝对路径。

将

> DirData=”/var/lib/awstats”

改为

> DirData = “/data/www.mytest.com/awstats” #这里为存放分析数据的路径,要保证这个目录存在，没有的话，自己手动创建一个

对应上边 Nginx 日志切割程序的所生成的目录存放结构，要注意 Awstats 的年月日格式的跟 Nginx 的写法有所不同。

我们现在执行统计的顺序是： Nginx 产生日志 –> 日志切割 –> Nginx 继续产生日志 –> 另存切割日志 –> 交由Awstats统计 –> 生成结果

在本文中 Awstats 所统计的日志，是已切下来的那部分。也能调转顺序，先统计完了再切。不过这比较容易造成统计的遗漏。配置修改完成后，保存退出。然后我们可以开始试一下手动执行。

**三.应用**

好了上面的教程基本上已经把awstats平台搭建完了，现在我们需要使用awstats来分析日志了。

 1. 先执行日志切割脚本 logcron.sh 把 Nginx 的日志切下来。
 2. 然后执行 Awstats 日志更新程序开始统计分析。

>

```
# /usr/local/etc/nginx/logcron.sh
# /usr/local/wwwroot/cgi-bin/awstats.pl -update -config=www.mytest.com

Create/Update database for config "/etc/awstats/awstats.www.mytest.com.conf" by AWStats version 7.0 (build 1.971)
From data in log file "/usr/local/etc/nginx/logs/access_20101220.log"...
Phase 1 : First bypass old records, searching new record...
Searching new records from beginning of log file...
Jumped lines in file: 0
Parsed lines in file: 10
 Found 0 dropped records,
 Found 0 comments,
 Found 0 blank records,
 Found 10 corrupted records,
 Found 0 old records,
 Found 0 new qualified records.
```

看到以上显示，证明日志切割和 Awstats 都已经运行无误了。统计分析完成后，结果还在 Awstats 的数据库中。在 Apache 上，可以直接打开 Perl 程序的网页查看统计。 但本文开始时已经提到，Nginx 对 Perl 支持并不好，所以我们要换个方法，利用 awstats 的工具将统计的结果生成静态文件，具体的步骤如下：

 * 首先在 /data/www.mytest.com 目录下创建一个文件夹web。例：/data/www.mytest.com/web
 * 然后让 Awstats 把静态页面生成到该目录中

>

```
# mkdir /data/www.mytest.com/web
# /usr/local/awstats/tools/awstats_buildstaticpages.pl -update -config=www.mytest.com -lang=cn -dir=/data/www.mytest.com/web -awstatsprog=/usr/local/wwwroot/cgi-bin/awstats.pl
```

上述命令的具体意思如下：

 * /usr/local/awstats/tools/awstats_buildstaticpages.pl Awstats 静态页面生成工具
 * -update -config=www.mytest.com 更新配置项
 * -lang=cn 语言为中文
 * -dir=/data/admin_web/awstats 统计结果输出目录
 * -awstatsprog=/usr/local/awstats/wwwroot/cgi-bin/awstats.pl Awstats 日志更新程序路径。

执行上面的命令后，在最后会提示一行信息，如：Main HTML page is ‘awstats.www.mytest.com.html’.

```
接下来，只需在nginx.conf 中，把该目录配置上去即可。 例子如下:(加粗部分):
```

>

```
server {
listen       80;
server_name  www.mytest.com;
```

>
> location / {
>
> index index.html index.htm index.php;
>
> }
>
>

```
location ~ ^/awstatsicons/ {             # 用别名访问图标目录
        alias   /usr/local/awstats/icons;
        index  index.html;
        access_log off;
        error_log off;
        charset gb2312;
        }
}
```

用浏览器查看到统计的详细结果  至此，使用 awstats 已能完全支持 Nginx 的日志统计。

为了让整个日志的统计过程自动完成，我们需要设置 crontab 计划任务，让 Nginx 日志切割以及 Awstats 自动运行，定时生成结果页面。

>

```
#vi /etc/crontab
59 11 * * * root /usr/local/etc/nginx/logcron.sh			#半夜11:59  进行日志切割
1 00 * * * root /usr/local/awstats/tools/awstats_buildstaticpages.pl -update -config=www.mytest.com -lang=cn -dir=/data/www.mytest.com/web -awstatsprog=/usr/local/wwwroot/cgi-bin/awstats.pl
```

```
#凌晨00:01  Awstats进行日志分析

:wq保存退出
```

**AWstats的安全设置**

一般站长都不愿随便让人知道自己站的真实流量，所以要把 Awstats 统计结果页面进行密码保护。Nginx 使用的是跟 Apache 一样的密码加密格式，这里需要用到 apache 自带的工具 htpasswd。 如果你在本机上默认装有 Apache，这你就只需在它的程序目录下运行 例:

>

```
#/usr/local/apache2/bin/htpasswd -c awstatspass admin #用户名为admin

New password:			输入密码
Re-type new password:		重复输入
Adding password for user admin	创建成功
```

```
然后把 awstatspass 这个密码包找个的地方藏起来.
修改 nginx.conf 在 location 中加入(加粗部分)：
```

>

```
server {
listen       80;
server_name  www.mytest.com;
```

>
> location / {
>
> index index.html index.htm index.php;
>
> }
>
> location /web/ {
>
> index index.html;
>
> access_log off;
>
> error_log off;
>
> charset gb2312;
>
> **auth_basic “admin”; #用户名**
>
> ****
>
> ****auth\_basic\_user_file　**/usr/local/etc/nginx/awstatspass; #密码包路径**
>
> }
>
>

```
;

        location /awstatsicons/ {             # 用别名访问图标目录
        alias   /usr/local/awstats/icons;
        index  index.html;
        access_log off;
        error_log off;
        charset gb2312;
        }
}
```

```
修改 Nginx 配置完毕后，执行命令 killall –s HUP nginx 让 Nginx 重新加载配置即可。

对于密码访问这一块本人没人进行测试的，请兴趣的可以试一下。
```