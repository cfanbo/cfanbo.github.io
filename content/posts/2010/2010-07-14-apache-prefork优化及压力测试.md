---
title: apache prefork优化及压力测试
author: admin
type: post
date: 2010-07-14T02:11:35+00:00
url: /archives/4637
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - 压力测试

---
优化apache prefork模式的参数, (384M内存openvz 的vps环境下面)

```
<IfModule mpm_prefork_module>
StartServers         12
MinSpareServers     12
MaxSpareServers     12
MaxClients         12
MaxRequestsPerChild 100
</IfModule>
```

```
StartServers是启动的进程数，Min和Max是最小最大进程数, MaxClients是最大可连接的客户端，MaxRequestPerChild是一个进程的生命周期内处理的请求数量，一旦达到设定的这个值，就回收进程。
```


这里的vps环境是内存384M最大可用，openvz的vps.其它优化设置可以参考

测试一千个客户端并发时的压力，可以用apache自带的ab.exe。

> ab -n 1000 -c 1000 http://www.netroby.com/index.php

**测试结果:**

```
Server Software:        Apache/2.2.14
Server Hostname:        www.netroby.com
Server Port:            80

Document Path:          /index.php
Document Length:        0 bytes

Concurrency Level:      1000
Time taken for tests:   281.953 seconds
Complete requests:      1000
Failed requests:        0
Write errors:           0
Non-2xx responses:      1000
Total transferred:      315000 bytes
HTML transferred:       0 bytes
Requests per second:    3.55 [#/sec] (mean)
Time per request:       281953.125 [ms] (mean)
Time per request:       281.953 [ms] (mean, across all concurrent requests)
Transfer rate:          1.09 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:      250  279 210.5    266    3281
Processing:  2109 138233 80553.2 136609  279156
Waiting:     2109 137264 81108.3 135359  279125
Total:       2375 138512 80561.8 136875  279750

Percentage of the requests served within a certain time (ms)
  50%  136875
  66%  184719
  75%  208547
  80%  221641
  90%  247469
  95%  266594
  98%  274438
  99%  277063
 100%  279750 (longest request)
```