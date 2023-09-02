---
title: '解决mac下brew link python3出错brew Error: Permission denied @ dir_s_mkdir – /usr/local/Frameworks'
author: admin
type: post
date: 2018-02-21T05:21:36+00:00
url: /archives/17599
categories:
 - 程序开发
tags:
 - python

---
mac上默认的python版本为2.7.10版本，需要升级到python3 版本，通过brew升级

```
$brew install python3
```

提示错误

> $ brew install python3 Warning: python3 3.6.3is already installed, it‘s just not linked. You can use \`brew link python3\` to link this version. $ brew link python3 Linking /usr/local/Cellar/python3/3.6.3… Error: Permission denied @ dir\_s\_mkdir

发现`/usr/local/`下没有路径`/usr/local/Frameworks`
需要新建该路径，并修改权限

解决：

```hljs ruby has-numbering
<span class="hljs-variable">$ </span>sudo mkdir /usr/local/<span class="hljs-constant">Frameworks</span>
<span class="hljs-variable">$ </span>sudo chown <span class="hljs-variable">$(</span>whoami)<span class="hljs-symbol">:admin</span> /usr/local/<span class="hljs-constant">Frameworks</span>
```

成功：

```hljs r has-numbering
$ brew link python3
Linking /usr/local/Cellar/python3/<span class="hljs-number">3.6</span><span class="hljs-number">.3</span><span class="hljs-keyword">...</span> <span class="hljs-number">1</span> symlinks created
```

参考：
[https://github.com/Homebrew/homebrew-core/issues/19286](https://github.com/Homebrew/homebrew-core/issues/19286)