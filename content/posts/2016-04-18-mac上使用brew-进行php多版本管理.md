---
title: Mac上使用Brew 进行PHP多版本管理
author: admin
type: post
date: 2016-04-18T13:33:16+00:00
url: /archives/16919
categories:
 - 程序开发
tags:
 - brew
 - php

---
[http://yansu.org/2014/09/26/use-old-version-of-brew-php.html](http://yansu.org/2014/09/26/use-old-version-of-brew-php.html)

## 版本切换方式 {#section}

通过brew安装的php可以通过 `brew link` 和 `brew unlink` 来切换不同版本。

例如

```
brew list
brew unlink php56
brew link php55

```

大版本可以用 `brew list` 来查，如果是小版本的话只能去 `/usr/local/Cellar/php55` 看了。这个时候使用 `php-version` 可以更方便一点。

我测试的此方法不行，只能使用php-verson 进行切换。

## 安装 `php-version` {#php-version}

[php-version](https://github.com/wilmoore/php-version) 是一个帮助管理从brew安装的php版本切换的工具。

安装非常简单

```
brew install php-version

```

然后执行

```
<span class="nb">source</span> <span class="k">$(</span>brew --prefix php-version<span class="k">)</span>/php-version.sh

```

## 使用 `php-version` {#php-version-1}

直接执行

```
php-version

```

就可以看到现有的版本，比如我自己的

```
<span class="gp">$ </span>php-version
  5.5.15
<span class="k">*</span> 5.5.16
  5.5.17

```

然后使用以下命令切换即可

```
php-version 5.5.15

```

再看php的版本，已经切换好了。

对于从默认mac自带的版本切换到新版本很方便，如

```
brew install php70 #最新版本php7.0.7
php-version 7.0.7
```