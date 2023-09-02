---
title: git checkout撤销未提交的修改
author: admin
type: post
date: 2013-08-02T10:03:24+00:00
url: /archives/14167
categories:
 - 程序开发
tags:
 - git

---
**(1) git checkout**
恢复某个已修改的文件（撤销未提交的修改）：

[shell]$ git checkout file-name[/shell]

例如：git checkout src/com/android/…/xxx.java

比如修改的都是java文件，不必一个个撤销，可以使用

[shell]$ git checkout *.java[/shell]

**撤销所有修改**

[shell]$ git checkout .[/shell]
更多参考： [http://hittyt.iteye.com/blog/1961386](http://hittyt.iteye.com/blog/1961386)