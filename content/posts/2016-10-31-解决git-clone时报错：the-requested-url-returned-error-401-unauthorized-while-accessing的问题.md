---
title: '解决git clone时报错：The requested URL returned error: 401 Unauthorized while accessing的问题'
author: admin
type: post
date: 2016-10-31T01:46:21+00:00
url: /archives/17133
categories:
 - 程序开发
tags:
 - git

---
版本问题，最直接的解决办法就是重新编辑安装git吧：

1. 下载：

```
wget -O git.zip https://github.com/git/git/archive/master.zip
```

2. 解压：

```
unzip git.zip
```

3. 进入git目录：

```
cd git-master
```

4. 编译安装：

```
autoconf
./configure --prefix=/usr/local
make && make install
```

5. 最后别忘了删掉旧的git，并把新版本的git建立软链接到/usr/bin/git

```
rm /usr/bin/git
ln -s /usr/local/bin/git /usr/bin/git
```