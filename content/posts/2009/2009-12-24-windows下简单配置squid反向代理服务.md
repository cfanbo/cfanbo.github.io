---
title: windows下简单配置squid反向代理服务…
author: admin
type: post
date: 2009-12-24T05:35:18+00:00
categories:
 - 服务器
tags:
 - Squid

---
下载windwosNT版本的squid下载地址：
http://squid.acmeconsulting.it/download/squid-2.6.STABLE13-bin.zip
1．把squid-2.6.STABLE13-bin.zip解压缩，把里面的squid文件夹拷到c:\下(squid默认的是c:\squid)
2．squid\etc目录下把
squid.conf.default拷贝一份重新命名为squid.conf
cachemgr.conf.default拷贝一份重新命名为cachemgr.conf
mime.conf.default拷贝一份重新命名为mime.conf
3．用文本编辑器打开squid.conf，需要修改的地方：
找到http_port 3128在后面增加一行
http_port 80 transparent
找到#cache_peer sib2.foo.net sibling 3128 3130 [proxy-only]在后面增加一行
cache_peer 192.168.1.8 parent 7001 0 no-query originserver
找到# TAG: visible_hostname在后面增加一行
visible_hostname volcano(任意命名)
找到http_access deny all在其前面加#将这一行注释掉，然后增加一行
http_access allow all
4．从命令行到c:\squid\sbin目录下执行
squid -i（将squid服务加入到服务里面）
squid -z
安装

完成

5．从服务里启动squid

访问squid服务器:

http://192.168.1.2(你的squid服务器IP地址)>>>指向http://192.168.1.8:7001(web服务器地址)

如果把#http_access deny all打开把http_access allow all注释掉，你的访问就会被拒绝

你需要配置一下：找到下面两行

#acl our_networks src 192.168.1.0/24 192.168.2.0/24 **#http_access allow our_networks**

**打开注释，修改你的内网ip(段)可以设为192.168.1.0/24一个也可以如上面的一样设一段IP**

**———————————————-**

**下载文件解压在c:\squid\，新建缓存d:\squid\var\cache,打开c:\squid\ext\**

**squid.conf.default拷贝一份重新命名为squid.conf**

**cachemgr.conf.default拷贝一份重新命名为cachemgr.conf**

**mime.conf.default拷贝一份重新命名为mime.conf**

**用写字板打开squid.conf**

**#http_port 3128 **http_port 192.168.29.149 80 transparent   #IP可写可不写，若出错不写，调试中注意任何占用80端口的程序，包括浏览器。****

**# none**

**cache_peer 192.168.29.150 parent 80 3100 no-query **acl allweb src 0.0.0.0/0.0.0.0cache_peer_access 192.168.29.150 allow allweb #所有请求都被转发到 192.168.29.150****

**# TAG: cache_peer_domain**

**# cache_mem 8 MB **cache_mem 1024 MB****

**# maximum_object_size_in_memory 8 KB **maximum_object_size_in_memory 2048 KB****

**# cache_dir ufs c:/squid/var/cache 100 16 256 **cache_dir aufs D:/squid/var/cache 10000 60 25   #10G****

**# maximum_object_size 4096 KB **maximum_object_size 500 MB****

**# cache_swap_low 90 **# cache_swap_high 95cache_swap_low 80cache_swap_high 95****

**refresh_pattern ^ftp:   1440 20% 10080 ignore-reload **refresh_pattern ^gopher: 1440 0% 1440 ignore-reload##refresh_pattern .   0 20% 4320#带问号的不缓存refresh_pattern \? 0 100% 0 ignore-reload#根结尾的url缓存10分钟refresh_pattern -i /$ 10 100% 10 ignore-reload#首页缓存10分钟refresh_pattern -i index\.(html|htm|php|jsp|do|aspx)$ 10 100% 10 ignore-reload#其他默认缓存1周refresh_pattern -i . 10080 100% 10080 ignore-reload****

**# none **visible_hostname My-Web-Cache               #名字随便取****

**# TAG: unique_hostname**