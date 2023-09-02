---
title: brew安装mysql为系统服务
author: admin
type: post
date: 2015-09-24T09:55:45+00:00
url: /archives/15999
categories:
 - 服务器
tags:
 - homebrew
 - mysql

---
默认情况下，使用brew install mysql 安装完mysql以后，需要手动启动mysql。为了方便以后使用，这里将mysql安装成系统服务。

`brew info mysql` 已经给了.plist文件，只需要load一下就可以，plist文件名不一定是`com.mysql.mysqld.plist`, 可以先到 \`brew –prefix mysql\` 目录看下。具体如下：

```
mkdir -p ~<span class="hljs-regexp">/Library/</span><span class="hljs-constant">LaunchAgents</span>
cp <span class="hljs-string">`brew --prefix mysql`</span>/com.mysql.mysqld.plist ~<span class="hljs-regexp">/Library/</span><span class="hljs-constant">LaunchAgents</span>/
launchctl load -w ~<span class="hljs-regexp">/Library/</span><span class="hljs-constant">LaunchAgents</span>/com.mysql.mysqld.plist

```

其实 [http://stackoverflow.com/questions/8014500/autostart-mysql-on-boot-from-terminal](http://stackoverflow.com/questions/8014500/autostart-mysql-on-boot-from-terminal) 这里有说。

 mkdir -p ~/Library/LaunchAgents
 cp `brew --prefix mysql`/*mysql*.plist ~/Library/LaunchAgents/
 launchctl load -w ~/Library/LaunchAgents/*mysql*.plist