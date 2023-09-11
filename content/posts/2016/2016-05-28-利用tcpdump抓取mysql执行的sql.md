---
title: 利用tcpdump抓取MySQL执行的SQL
author: admin
type: post
date: 2016-05-28T03:41:05+00:00
url: /archives/17030
categories:
 - 程序开发
tags:
 - tcpdump
 - 抓包

---
[http://ourmysql.com/archives/1358](http://ourmysql.com/archives/1358)
编写脚本文件dumpsql.sh,内容如下：

```
#!/bin/bash
#this script used montor mysql network traffic.echo sql
tcpdump -i eth0 -s 0 -l -w - dst port 3306 | strings | perl -e '
while(<>) { chomp; next if /^[^ ]+[ ]*$/;
    if(/^(SELECT|UPDATE|DELETE|INSERT|SET|COMMIT|ROLLBACK|CREATE|DROP|ALTER|CALL)/i)
    {
        if (defined $q) { print "$q\n"; }
        $q=$_;
    } else {
        $_ =~ s/^[ \t]+//; $q.=" $_";
    }
}

```

运行并抓去sql的执行。

抓取后在当前目录出现out.log文件，执行strings out.log即可看到sql的运行情况