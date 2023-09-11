---
title: 使用 Portmaster 升级 Ports
author: admin
type: post
date: 2011-01-02T08:29:25+00:00
url: /archives/7433
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - portmaster

---
**Portmaster** 是另外一个用来升级已安装的 ports 的工具。 **Portmaster** 被设计成尽可能使用 “基本” 系统中能找到的工具 （它不依赖于其他的 ports） 和 `/var/db/pkg/` 中的信息来检测出需要升级的 ports。你可以在 [ `ports-mgmt/portmaster`][1] 找到它：

```
# cd /usr/ports/ports-mgmt/portmaster
# make install clean
```

**Portmaster** groups ports into four categories:

**Portmaster** 把 ports 分成4类：

 * Root ports (不依赖其他的 ports，也不被依赖)
 * Trunk ports (不依赖其他的 ports，但是被其他的 ports 依赖)
 * Branch ports (依赖于其他的 ports，同时也被依赖)
 * Leaf ports (依赖于其他的 ports，但不被依赖)

你可以使用 `-L` 选项列出所有已安装的 ports 和查找存在更新的 ports：

```
# portmaster -L
===>>> Root ports (No dependencies, not depended on)
===>>> ispell-3.2.06_18
===>>> screen-4.0.3
        ===>>> New version available: screen-4.0.3_1
===>>> tcpflow-0.21_1
===>>> 7 root ports
...
===>>> Branch ports (Have dependencies, are depended on)
===>>> apache-2.2.3
        ===>>> New version available: apache-2.2.8
...
===>>> Leaf ports (Have dependencies, not depended on)
===>>> automake-1.9.6_2
===>>> bash-3.1.17
        ===>>> New version available: bash-3.2.33
...
===>>> 32 leaf ports

===>>> 137 total installed ports
        ===>>> 83 have new versions available
```

可以使用这个简单的命令升级所有已安装的 ports：

```
# portmaster -a
```

> **注意:** **Portmaster** 默认在删除一个现有的 port 前会做一个备份包。如果新的版本能够被成功安装， **Portmaster** 将删除备份。 使用 `-b` 后 **Portmaster** 便不会自动删除备份。加上 `-i` 选项之后 **Portmaster** 将进入互动模式， 在升级每个 port 以前提示你给予确认。

如果你在升级的过程中发现了错误，你可以使用 `-f` 选项升级/重新编译所有的 ports：

```
# portmaster -af
```

同样你也可以使用 **Portmaster** 往系统里安装新的 ports，升级所有的依赖关系之后并安装新的 port：

```
# portmaster shells/bash
```

更多的详细信息请参阅 [portmaster(8)][2]

 [1]: http://www.freebsd.org/cgi/url.cgi?ports/ports-mgmt/portmaster/pkg-descr
 [2]: http://www.freebsd.org/cgi/man.cgi?query=portmaster&sektion=8&manpath=FreeBSD+7.0-RELEASE+and+Ports