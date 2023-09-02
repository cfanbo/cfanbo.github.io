---
title: 使用supervisord监控php脚本
author: admin
type: post
date: 2016-11-09T12:20:36+00:00
url: /archives/17153
categories:
 - 程序开发
tags:
 - Supervisord

---
官网： [http://www.supervisord.org](http://www.supervisord.org)

一、安装supervisord

```
$brew install supervisord
```

在mac下安装要比linux下安装方便的多。

二、配置

修改/usr/local/etc/supervisord.ini文件，取消以下几行注释

```
[inet_http_server] ; inet (TCP) server disabled by default
port=127.0.0.1:9001 ; (ip_address:port specifier, *:port for all iface)
username=user ; (default is no username (open server))
password=123 ; (default is no password (open server))

```

这样就可以通过浏览器对进行管理了。

三、添加一个新应用

创建一个a.php文件，内容如下：

```
while(true){
echo 'a' . time() . "\r\n";
sleep(1);
}
```

然后在supervisord.ini文件中添加以下几行：

```
[program:php]
command=php /Users/sxf/web/a.php
autostart=true
autorestart=true
startsecs=1
startretries=3
redirect_stderr=true
stdout_logfile=/Users/sxf/web/supervisord.log
stderr_logfile=/Users/sxf/web/stderr.log

```

重启supervisord。

```
$brew services restart supervisord
```

打开浏览器 [http://127.0.0.9001](http://127.0.0.9001),输入用户名和密码，可以看此进程,可以对每个进程进行停止，重启和刷新操作。

[![supervisord_php](http://blog.haohtml.com/wp-content/uploads/2016/11/supervisord_php.png)][1]
对于supervisord命令请参考： [http://blog.haohtml.com/archives/15145](http://blog.haohtml.com/archives/15145)

 [1]: http://blog.haohtml.com/wp-content/uploads/2016/11/supervisord_php.png