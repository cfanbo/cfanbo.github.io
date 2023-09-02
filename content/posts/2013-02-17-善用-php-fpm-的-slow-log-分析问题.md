---
title: 善用 PHP-FPM 的 slow log 分析问题
author: admin
type: post
date: 2013-02-17T04:18:55+00:00
url: /archives/13659
categories:
 - 服务器
tags:
 - php-fpm

---
节前公司站点出现了莫名的 502 错误，在服务器配置上拆腾未果，重新开始怀疑程序问题。

关于 502 错误，具体可以参考以下两篇文章：
[《自动检测 PHP-FPM 的错误并重启的 PHP 脚本》][1]
[《NGINX + PHP-FPM 502 相关事》][2]

根据错误提示(11: Resource temporarily unavailable) ，排除掉服务器配置的问题，自然而然就怀疑是资源被程序占用光了。

这些资源包括数据库连接、文件数、锁等等，如果一个个去猜解调试甚至是走读代码，未免太费时间，也未必能发现问题所在。

好在 PHP-FPM 提供了慢执行日志，可以将执行比较慢的脚本的调用过程 dump 到日志中。

配置比较简单，PHP 5.3.3 之前设置如下：

```
The timeout (in seconds) for serving of single request after which a php backtrace will be dumped to slow.log file
'0s' means 'off'
<value name="request_slowlog_timeout">1s</value>

The log file for slow requests
<value name="slowlog">logs/slow.logs</value>
```

PHP 5.3.3 之后设置以下如下：

```
; The timeout for serving a single request after which a PHP backtrace will be
; dumped to the 'slowlog' file. A value of '0s' means 'off'.
; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
; Default Value: 0
request_slowlog_timeout = 1s

; The log file for slow requests
; Default Value: /usr/local/php/log/php-fpm.log.slow
slowlog = /usr/local/php/log/php-fpm.log.slow
```

还可以将执行时间太长的进程直接终止，设置下执行超时时间即可。
PHP 5.3.3 之前版本：

```
The timeout (in seconds) for serving a single request after which the worker process will be terminated
Should be used when 'max_execution_time' ini option does not stop script execution for some reason
'0s' means 'off'
<value name="request_terminate_timeout">10s</value>
```

PHP 5.3.3+：

```
; The timeout for serving a single request after which the worker process will
; be killed. This option should be used when the 'max_execution_time' ini option
; does not stop script execution for some reason. A value of '0' means 'off'.
; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
; Default Value: 0
request_terminate_timeout = 10s
```

加上慢执行日志后，我们可以很容易从慢执行日志中看出问题所在，比如：

```
Feb 07 19:00:30.378095 pid 27012 (pool default)
script_filename = /www/adshow/a.php
[0x000000000115ea08] flock() /www/backend/parser/logs.class.php:260
[0x0000000001159810] lock_stats() /www/adshow/a.php:126

Feb 07 19:00:31.033073 pid 27043 (pool default)
script_filename = /www/adshow/a.php
[0x00000000012686e8] flock() /www/backend/parser/logs.class.php:260
[0x00000000012634f0] lock_stats() /www/adshow/a.php:126

Feb 07 19:00:31.163995 pid 26979 (pool default)
script_filename = /www/adshow/a.php
[0x000000000115bda8] flock() /www/backend/parser/logs.class.php:260
[0x0000000001156bb0] lock_stats() /www/adshow/a.php:126

Feb 07 19:00:31.164102 pid 27033 (pool default)
script_filename = /www/adshow/a.php
[0x00000000013f3fa8] flock() /www/backend/parser/logs.class.php:260
[0x00000000013eedb0] lock_stats() /www/adshow/a.php:126

Feb 07 19:00:31.297313 pid 27448 (pool default)
script_filename = /www/adshow/a.php
[0x0000000001180218] flock() /www/backend/parser/logs.class.php:260
[0x000000000117b020] lock_stats() /www/adshow/a.php:126
```

很明显是程序中产生了死锁，导致各个 PHP-CGI 进程互相等待资源而锁死。
据此，再进行进一步的程序分析，就更具方向性了。

日志说明：
script_filename 是入口文件
flock() ： 说明是执行这个方法的时候超过执行时间的。
lock\_stats() ：说明调用flock()的方法是flock\_stats() 。
每行冒号后面的数字是行号。

– EOF –

转自： [http://hily.me/blog/2011/02/use-php-fpm-slow-log-to-investigate-problem/](http://hily.me/blog/2011/02/use-php-fpm-slow-log-to-investigate-problem/)

 [1]: http://hily.me/blog/2011/01/php-fpm-auto-restart-on-error-occur/
 [2]: http://hily.me/blog/2011/01/nginx-php-fpm-502/