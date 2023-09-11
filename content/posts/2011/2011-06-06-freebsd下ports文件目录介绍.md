---
title: FreeBSD下Ports文件目录介绍
author: admin
type: post
date: 2011-06-06T15:07:42+00:00
url: /archives/9680
IM_contentdowned:
 - 1
categories:
 - 服务器

---
当提到 Ports Collection 时， 第一个要说明的就是何谓 “skeleton”。 简单地说， port skeleton 是让一个程序在 FreeBSD 上简洁地编译并安装的所需文件的最小组合。 每个 port skeleton 包含：

 * 一个 `Makefile`。 `Makefile` 包括好几个部分， 指出应用程序是如何编译以及将被安装在系统的哪些地方。
 * 一个 `distinfo` 文件。这个文件包括这些信息： 这些文件用来对下载后的文件校验和进行检查 (使用 [sha256(1)][1])， 来确保在下载过程中文件没有被破坏。
 * 一个 `files` 目录。 这个目录包括在 FreeBSD 系统上编译和安装程序需要用到的补丁。 这些补丁基本上都是些小文件， 指出特定文件作了哪些修正。 它们都是纯文本的的格式，基本上是这样的 “删除第 10 行” 或 “将第 26 行改为这样 …”， 补丁文件也被称作 “diffs”， 他们由 [diff(1)][2]程序生成。
 这个目录也包含了在编译 port 时要用到的其它文件。

 * 一个 `pkg-descr` 文件。 这是一个提供更多细节，有软件的多行描述。
 * 一个 `pkg-plist` 文件。 这是即将被安装的所有文件的列表。它告诉 ports 系统在卸载时需要删除哪些文件。

 一些ports还有些其它的文件， 例如 `pkg-message`。 ports 系统在一些特殊情况下会用到这些文件。 如果您想知道这些文件更多的细节以及 ports 的概要， 请参阅 [FreeBSD Porter’s Handbook](http://docs.haohtml.com/doc/en_US.ISO8859-1/books/porters-handbook/index.html)。

 port里面包含着如何编译源代码的指令， 但不包含真正的源代码。 您可以在网上或 CD-ROM 上获得源代码。 源代码可能被开发者发布成任何格式。 一般来说应该是一个被 tar 和 gzip 过的文件， 或者是被一些其他的工具压缩或未压缩的文件。 ports中这个程序源代码标示文件叫 “distfile”， 安装 FreeBSD port的方法还不止这两种。

 摘自：



 [1]: http://www.freebsd.org/cgi/man.cgi?query=sha256&sektion=1
 [2]: http://www.freebsd.org/cgi/man.cgi?query=diff&sektion=1