---
title: git init 和git –bare init 的具体区别？
author: admin
type: post
date: 2011-12-11T10:35:50+00:00
url: /archives/12265
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - git

---
一般个人使用，用git init，这时候你的工作区也在这里。你要是想建立一个固定的地址让大家一起用，就在服务器上用git –bare init。

其实你可以看到，init建立的.git目录内容和–bare建立的目录内容是差不多的。

**在初始化远程仓库时最好使用 git –bare init   而不要使用：git init。这样在使用hooks的时候，会有用处。**

如果使用了git init初始化，则远程仓库的目录下，也包含work tree，当本地仓库向远程仓库push时,   如果远程仓库正在push的分支上（如果当时不在push的分支，就没有问题）, 那么push后的结果不会反应在work tree上,  也即在远程仓库的目录下对应的文件还是之前的内容，必须得使用git reset –hard才能看到push后的内容.