---
title: 修改gitlab的项目部署url地址
author: admin
type: post
date: 2016-05-09T04:24:07+00:00
url: /archives/16940
categories:
 - 服务器
tags:
 - gitlab

---
<刚在centos7上部署了最新版本的gitlab 8.7.3，发现在创建的git项目地址为http://localhost/username/www.git ，外网无法访问，这时我们只需要修改一个配置文件即可。我安装的时候全部使用的默认配置，路径为 /var/opt/gitlab/gitlab-rails/etc/，配置文件为 gitlab.yml ,文件顶部配置如下：

```
host: localhost
port: 80
https: false
```

修改host值为你想使用的外网域名或服务器IP地址即可，保存退出。

```
gitlab-ctl restart
```

注意这里的命令是restart不是 reconfigure(根据配置文件/etc/gitlab/gitlab.rb 重新生成配置) ，否则还会恢复原来的配置。

用ps -ef | grep nginx命令看了下，发现运行的nginx的路径是/opt/gitlab/embedded/sbin/nginx，而配置文件路径是/var/opt/gitlab/nginx，怪不得我打开/etc/nginx/nginx.conf没看到gitlab相关的配置。
试着改了下/var/opt/gitlab/nginx/nginx.conf 和 /var/opt/gitlab/nginx/gitlab-http.conf，重启服务后，页面无法访问了，我先折腾一下。
但配置文件为/etc/gitlab/gitlab.rb来配置

===============

## 如何修改gitlab监听端口

1.修改gitlab监听端口，默认为使用80端口，有时服务器中已经有nginx/apache占用了这个端口，修改监听端口来避免此问题
修改 /etc/gitlab/gitlab.rb 文件，修改下行

```
# nginx['listen_port'] = nil
```

为

```
nginx['listen_port'] = 8089
```

保存，执行重新配置命令

```
sudo gitlab-ctl reconfigure
```

通过浏览器 http://ip:8089 即可访问。

2.如何让gitlab只监听本机内网_ip_

修改

```
# nginx['listen_addresses'] = ['*']
```

为

```
nginx['listen_addresses']= ['127.0.0.1']
```

## 如何备份与恢复gitlab

[https://segmentfault.com/a/1190000002439923](https://segmentfault.com/a/1190000002439923)