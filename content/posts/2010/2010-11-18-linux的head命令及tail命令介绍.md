---
title: linux的head命令及tail命令介绍
author: admin
type: post
date: 2010-11-18T03:09:14+00:00
url: /archives/6692
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - linux的

---
当需要查看一个文本文件的头部或尾部时，head 命令及tail 命令可以非常方便的完成该操作。head 命令用于查看一个文本文件的开头部分；而tail 命令则用于显示文本文件的末尾几行。这两个命令举例如下：
head example.txt 显示文件 example.txt 的前十行内容；
head -n 20 example.txt 显示文件 example.txt 的前二十行内容；
tail example.txt 显示文件 example.txt 的后十行内容；
tail -n 20 example.txt 显示文件 example.txt 的后二十行内容；
tail -f example.txt 显示文件 example.txt 的后十行内容并在文件内容增加后，自动显示新增的文件内容。
注意：
最后一条命令非常有用，尤其在监控日志文件时，可以在屏幕上一直显示新增的日志信息。