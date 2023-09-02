---
title: git无法pull仓库refusing to merge unrelated histories的解决办法
author: admin
type: post
date: 2017-03-29T02:29:28+00:00
url: /archives/17414
categories:
 - 程序开发
tags:
 - git

---
在本地有一个很久的项目，需要上传到远程git仓库中，先在远程创建了一个仓库，在本地配置好后，发现上传的时候，提示先git pull 一下，但git pull的时候提示这个错误，主要是因为目前两个git仓库信息不一致问题，我们要将两个项目合并在一起，需要添加一个  –allow-unrelated-histories 参数即可。

先进入cli命令行模式，执行

```
git pull origin master --allow-unrelated-histories
```

然后再执行git pull origin master 即可。

参考：