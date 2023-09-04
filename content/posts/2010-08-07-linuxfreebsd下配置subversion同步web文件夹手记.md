---
title: Linux/FreeBSD下配置Subversion同步Web文件夹手记
author: admin
type: post
date: 2010-08-07T02:38:11+00:00
url: /archives/4990
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - svn

---

**下载安装subversion-1.6.9.tar.gz**

1> svnserve配置

1.1 创建仓库存放目录

mkdir -p /opt/svn/repos

chown -R svn:svn /opt/svn/

2> 启动svn

svnserve -d –listen-host=0.0.0.0 –listen-port=3190 -r /data/repos

3> 创建SVN仓库

svnadmin create /opt/svn/repos/abc

vim /opt/svn/repos/abc/conf/passwd

4> 新增访问用户名和密码

格式如下

[users]

roger=123456＃用户名=密码

5> 修改 svnserve.conf

#vi /opt/svn/repos/abc/conf/svnserve.conf

#password-db = passwd为password-db = passwd //使用密码文件

#anon-access = read 为 anon-access = read //匿名可以读取,如果设置必须输入密码才能读则修改read为none

#auth-access = write 为 auth-access = write //信任用户可写

6> 迁出到要更新的web目录

svn checkout file:///opt/svn/repos/abc /www/abc //牵出到服务器abc目录

svn update file:///opt/svn/repos/abc /www/abc //手动更新到服务器abc目录

7> 设置自动更新abc目录

拷贝/opt/svn/repos/abc中hooks下的post-commit.tmpl为post-commit

cp post-commit.tmpl post-commit

chmod 777 post-commit

并修改post-commit中的

REPOS=”$1″

REV=”$2″

commit-email.pl “$REPOS” “$REV” commit-watchers@example.org

log-commit.py –repository “$REPOS” –revision “$REV”

为：

export LANG=en_US.UTF-8 #中文文件问题

svn update file:///opt/svn/repos/abc –username root –password 123456 /www/abc

其中 file:///opt/svn/repos/abc改成你实际的svn库的位置 /www/abc改成你实际的web目录

8> 备份一个版本库

svnsync是Subversion的远程版本库镜像工具，它允许你把一个版本库的内容录入到另一个。

svnsync init file:///opt/svn/repos_dest file:///opt/svn/repos_source

svnsync sync file:///opt/svn/repos_dest