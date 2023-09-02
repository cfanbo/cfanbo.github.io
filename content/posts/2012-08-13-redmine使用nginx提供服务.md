---
title: redmine使用nginx提供服务
author: admin
type: post
date: 2012-08-13T10:55:42+00:00
url: /archives/13289
categories:
 - 程序开发
tags:
 - redmine

---
上一节 [http://blog.haohtml.com/archives/13282](http://blog.haohtml.com/archives/13282) 我们介绍了在centos下安装redmine软件的方法，但使用时候需要使用ip:3000　的形式才可以访问，不是太方便，我们习惯使用域名的形式来处理的。这里我们直接使用域名redmine.haohtml.com 来访问. 我们使用的是web server　为 nginx 。

我们使用虚拟主机配置文件redmine.conf.内容如下：

```
upstream mongrel{
server 127.0.0.1:3000;
}

server {

listen 80;
server_name redmine.haohtml.com;
root /data/wwwroot/redmine/redmine-2.0.3/public;
location / {
index index.php index.html index.shtml;
proxy_pass http://mongrel;
proxy_redirect off;
proxy_set_header Host $host;
proxy_set_header X-Real_IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}

#log...

location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|js|css)$
{
expires 30d;
}

}

```

然后执行命令

```
#/usr/local/nginx/sbin/nginx -t
#/usr/local/nginx/sbin/nginx -s reload
```

这里通过域名 redmine.haohtml.com　就可以访问到redmine了。

**注意：**

由于以前启动过redmine，使用的端口为3000.在操作以前需要将ruby进程结束一下，不然会提示tcp server已经被使用了。然后再执行

```
ruby script/rails server webrick -e production
```

就可以了。

不过这里并没有限制直接通过ip:3000 的访问方法的。如果限制，可以使用iptables来实现。

**相关教程：**

centos安装redmine项目管理系统[教程]: [http://blog.haohtml.com/archives/13282](http://blog.haohtml.com/archives/13282)
Redmine局域网访问缓慢问题解决: [http://blog.haohtml.com/archives/13272](http://blog.haohtml.com/archives/13272)