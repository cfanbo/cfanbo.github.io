---
title: php中的$_SERVER or $_ENV
author: admin
type: post
date: 2009-05-25T02:33:43+00:00
excerpt: $_SERVER 是一个包含诸如头信息（header）、路径（path）和脚本位置（script locations）的数组。数组的实体由 web 服务器创建。不能保证所有的服务器都能产生所有的信息；服务器可能忽略了一些信息，或者产生了一些未在下面列出的新的信息。这意味着，大量的这些变量在 CGI 1.1 规范中说明，所以应该仔细研究一下。
url: /archives/1499
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php

---
服务器变量：$_SERVER
注: 在 PHP 4.1.0 及以后版本使用。之前的版本，使用 $HTTP\_SERVER\_VARS。

$_SERVER 是一个包含诸如头信息（header）、路径（path）和脚本位置（script locations）的数组。数组的实体由 web 服务器创建。不能保证所有的服务器都能产生所有的信息；服务器可能忽略了一些信息，或者产生了一些未在下面列出的新的信息。这意味着，大量的这些变量在 CGI 1.1 规范中说明，所以应该仔细研究一下。



环境变量：$_ENV
注: 在 PHP 4.1.0 及以后版本使用。之前的版本，使用 $HTTP\_ENV\_VARS。

在解析器运行时，这些变量从环境变量转变为 PHP 全局变量名称空间（namespace）。它们中的许多都是由 PHP 所运行的系统决定。完整的列表是不可能的。请查看系统的文档以确定其特定的环境变量。

其它环境变量（包括 CGI 变量），无论 PHP 是以服务器模块或是以 CGI 处理方式运行，都在这里列出了。

这是一个“superglobal”，或者可以描述为自动全局变量。这只不过意味这它在所有的脚本中都有效。在函数或方法中不需要使用 global $\_ENV; 来访问它，就如同使用 $HTTP\_ENV_VARS 一样。

$HTTP\_ENV\_VARS 包含着同样的信息，但是不是一个自动全局变量（注意：$HTTP\_ENV\_VARS 和 $_ENV 是不同的变量，PHP 处理它们的方式不同）。

如果设置了 register\_globals 指令，这些变量也在所有脚本中可用；也就是，分离了 $\_ENV 和 $HTTP\_ENV\_VARS 数组。相关信息，请参阅安全的相关章节使用 Register Globals。这些单独的全局变量不是自动全局变量。