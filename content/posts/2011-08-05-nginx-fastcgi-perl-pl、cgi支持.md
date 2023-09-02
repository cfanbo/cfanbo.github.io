---
title: Nginx fastcgi perl (pl、cgi)支持
author: admin
type: post
date: 2011-08-05T15:57:22+00:00
url: /archives/10909
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - fastcgi
 - nginx
 - perl

---
**1. 安装FCGI模块**

> \# wget http://search.cpan.org/CPAN/authors/id/B/BO/BOBTFISH/FCGI-0.70.tar.gz
> \# tar zxvf FCGI-0.70.tar.gz
> \# cd FCGI-0.70
> \# perl Makefile.PL
> \# make
> \# make install

**2. 安装 IO 和 IO::ALL模块**

> \# wget http://search.cpan.org/CPAN/authors/id/G/GB/GBARR/IO-1.25.tar.gz
> \# tar zxvf IO-1.25.tar.gz
> \# cd IO-1.25
> \# perl Makefile.PL
> \# make
> \# make install
>
>

> **升级MakeMaker版**
>

>
>

> #wget http://search.cpan.org/CPAN/authors/id/M/MS/MSCHWERN/ExtUtils-MakeMaker-6.54.tar.gz

#tar zxvf  ExtUtils-MakeMaker-6.54
>

>
>

> # perl Makefile.PL
>

>
>

> Checking if your kit is complete…
>

>
>

> Looks good
>

>
>

> Using included version of ExtUtils::Manifest (1.56) as it is newer than the installed version (1.46).
>

>
>

> Using included version of ExtUtils::Command (1.16) as it is newer than the installed version (1.09).
>

>
>

> Using included version of ExtUtils::Installed (1.43) as it is newer than the installed version (0.08).
>

>
>

> Using included version of ExtUtils::Packlist (1.43) as it is newer than the installed version (0.04).
>

>
>

> Using included version of ExtUtils::Install (1.52) as it is newer than the installed version (1.33).
>

>
>

> Writing Makefile for ExtUtils::MakeMaker
>

>
>

> make
>

>
>

> make install
>

>
> \# wget http://search.cpan.org/CPAN/authors/id/I/IN/INGY/IO-All-0.41.tar.gz
> \# tar zxvf IO-All-0.41.tar.gz
> \# cd IO-All
> \# perl Makefile.PL
> \# make
> \# make install

**3. Perl脚本**

[点击下载Perl脚本perl-fcgi][1]

我把这个脚本放在 /usr/local/nginx/perl-fcgi.pl

> \# chmod 755 /usr/local/nginx/perl-fcgi.pl

**4.cgi启动/停止脚本** (nobody为nginx的运行用户)

\# vi /usr/local/nginx/start\_perl\_cgi.sh

```
#!/bin/bash
#set -x
dir=/usr/local/nginx

stop ()
{
#pkill  -f  $dir/perl-fcgi.pl
kill $(cat $dir/logs/perl-fcgi.pid)
rm $dir/logs/perl-fcgi.pid 2>/dev/null
rm $dir/logs/perl-fcgi.sock 2>/dev/null
echo "stop perl-fcgi done"
}

start ()
{
rm $dir/now_start_perl_fcgi.sh 2>/dev/null

chown nobody.root $dir/logs
echo "$dir/perl-fcgi.pl -l $dir/logs/perl-fcgi.log -pid $dir/logs/perl-fcgi.pid -S $dir/logs/perl-fcgi.sock" >>$dir/now_start_perl_fcgi.sh

chown nobody.nobody $dir/now_start_perl_fcgi.sh
chmod u+x $dir/now_start_perl_fcgi.sh

sudo -u nobody $dir/now_start_perl_fcgi.sh
echo "start perl-fcgi done"
}

case $1 in
stop)
stop
;;
start)
start
;;
restart)
stop
start
;;
esac
```

> \# chmod 755 /usr/local/nginx/start\_perl\_cgi.sh

\# 启动脚本

> \# /usr/local/nginx/start\_perl\_cgi.sh start

正常情况下在/usr/local/nginx/logs 下生成 perl-fcgi.sock 这个文件,如果没有生成,那就要检查下上面的步聚了.

**5. 配置nginx**

```
server {
    listen       80;
    server_name  _;
    location / {
        root   /usr/local/nginx/html;
        index  index.html index.htm index.cgi;
    }
    location ~ .*\.(pl|cgi)?$
    {
      gzip off;
      root   /data/nginx/html;
      fastcgi_pass  unix:/usr/local/nginx/logs/perl-fcgi.sock;
      fastcgi_index index.cgi;
      fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
      fastcgi_param  SERVER_SOFTWARE    nginx;
      fastcgi_param  QUERY_STRING       $query_string;
      fastcgi_param  REQUEST_METHOD     $request_method;
      fastcgi_param  CONTENT_TYPE       $content_type;
      fastcgi_param  CONTENT_LENGTH     $content_length;
      fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
      fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
      fastcgi_param  REQUEST_URI        $request_uri;
      fastcgi_param  DOCUMENT_URI       $document_uri;
      fastcgi_param  DOCUMENT_ROOT      $document_root;
      fastcgi_param  SERVER_PROTOCOL    $server_protocol;
      fastcgi_param  REMOTE_ADDR        $remote_addr;
      fastcgi_param  REMOTE_PORT        $remote_port;
      fastcgi_param  SERVER_ADDR        $server_addr;
      fastcgi_param  SERVER_PORT        $server_port;
      fastcgi_param  SERVER_NAME        $server_name;
      fastcgi_read_timeout   60;
     }
}
```

**6.Perl探针测试**

[点击下载Perl探针][2]

转载:

 [1]: http://blog.haohtml.com/wp-content/uploads/2011/08/perl-fcgi.zip
 [2]: http://www.xtgly.com/wp-content/uploads/2010/09/perlinfo.zip