---
title: mac手动停止 php-fpm 服务
author: admin
type: post
date: 2019-07-01T04:39:10+00:00
url: /archives/18977
categories:
 - 其它
tags:
 - php-fpm

---
由于要安装一个docker服务，对外提供端口用的是9000, 和php-fpm的监听端口冲突，所以需要先停止一下php-fpm服务。

多次执行

`sudo killall php-fpm`

发现过一会php-fpm会自动启动，就算一个一个的进程kill -9 也一样的效果。经过分析这个应该是和php-fpm配置文件 ~/Library/LaunchAgents/homebrew.mxcl.php@7.1.plist 有关。

我们知道 ~/Library/LaunchAgents 针对当前用户的启动项目录，针对这个项目里的一些配置服务有一个 launchctl 命令可以操作，其中有几个命令我们需要知道他的意思

```
launchctl load 启动plist运行
launchctl unload  卸载
launchctl list 查看所有启动任务
```

默认当用户登录后，mac系统会对当前目录 ~/Library/LaunchAgents 里的每个配置服务文件自动执行launchctl load 命令。如果我们想停止一个服务的话，则需要执行 launchctl unload 命令即可。

`$ LaunchAgents launchctl list | grep php

 66054 0 homebrew.mxcl.php@7.1

$ LaunchAgents launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.php@7.1.plist

$ LaunchAgents launchctl list | grep php`

然后再用ps 查看确认php-fpm已停止。