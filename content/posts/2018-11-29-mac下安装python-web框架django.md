---
title: mac下安装python web框架django
author: admin
type: post
date: 2018-11-29T06:23:55+00:00
url: /archives/18612
categories:
 - 程序开发
tags:
 - django
 - python

---
**前提**

由于mac自带的python2.7（路径 /usr/bin/python）
后来手动又安装了python3.7（/usr/local/bin/python3）

两个版本共存。为了方便，直接在.zshrc文件里做了别名映射

```
alias python="/usr/local/bin/python3.7"
```

所以直接使用命令python实际上用的是3.7版本。

按照官方教程 [https://docs.djangoproject.com/zh-hans/2.1/intro/install/](https://docs.djangoproject.com/zh-hans/2.1/intro/install/) 安装django。发现在使用命令 pip install django 安装后发现检测不到django，很奇怪，后来发现了问题所在。

```
➜  ~ python
Python 3.7.1 (default, Nov  6 2018, 18:46:03)
[Clang 10.0.0 (clang-1000.11.45.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import django
Traceback (most recent call last):
  File "", line 1, in
ModuleNotFoundError: No module named 'django'
```

原来python2对应一个是包安装命令是 pip，而python3对应的是一个叫做pip3的命令。所以使用新的命令

```
pip3 install django
```

安装成功。

```
➜  ~ python
Python 3.7.1 (default, Nov  6 2018, 18:46:03)
[Clang 10.0.0 (clang-1000.11.45.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import django
>>> print(django.get_version())
2.1.3

```