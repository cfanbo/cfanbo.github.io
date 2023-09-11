---
title: Troubleshooting varnish（varnish排错方法）
author: admin
type: post
date: 2011-11-20T21:30:52+00:00
url: /archives/12055
IM_contentdowned:
 - 1
categories:
 - 系统架构

---
1.有时候 varnish 会出错，为了使您知道该检查哪里，您可以检查 varnishlog，/var/log/syslog/,var/log/messages 这里可以发现一些信息，知道varnish怎么了。
****

2.When varnish won’t start
有些时候，varnish 不能启动。这里有很多 varnish不能启动的原因，通常我们可以观看/dev/null的权限和是否其他软件占用了端口。
使用debug模式启动 varnish，然后观看发生了什么：

```
varnishd  -f /usr/local/etc/varnish/default.vcl  -s malloc,1G  -T 127.0.0.1:2000   -a 0.0.0.0:8080 –d
```

提示-d 选项，它将给您更多的信息关于接下来发生了什么。让我们看看如果其他程序暂用了varnish 的端口，它将显示什么：

```
# varnishd  -n foo  -f /usr/local/etc/varnish/default.vcl  -s malloc,1G  -T 127.0.0.1:2000
-a 0.0.0.0:8080 -d
storage_malloc: max size 1024 MB.
Using old SHMFILE
Platform: Linux,2.6.32-21-generic,i686,-smalloc,-hcritbit
200 193
-----------------------------
Varnish HTTP accelerator CLI.
-----------------------------
Type 'help' for command list.
Type 'quit' to close CLI session.
Type 'start' to launch worker process.
```

现在 varnish 的主程序已经运行，在 debug 模式中，cache 现在还没有启动，现在
您在终端中使用“start”命令来让主程序开启 cache功能

>

```
start
bind(): Address already in use
300 22
Could not open sockets
```

在这里，我们发现一个问题。Varnish要使用的端口被 HTTP使用了。
3.Varnish is creashing
当varnish宕掉的时候。

4.Varnish gives me guru mediation
首先查找varnishlog，这里可能会给您一些信息。

5.Varnish doesn’t cache
请参考“提高命中率”这章。