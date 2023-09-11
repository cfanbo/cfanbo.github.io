---
title: '500 OOPS: vsftpd: refusing to run with writable anonymous root'
author: admin
type: post
date: 2008-10-29T13:11:48+00:00
excerpt: |
 如果我们已经把vsFTPd服务器启动好了，但登录测试是会出现类似下面的提示；

 500 OOPS: vsftpd: refusing to run with writable anonymous root

 这表示ftp用户的家目录的权限不对，应该改过才对；

 [root@localhost ~]# more /etc/passwd |grep ftp
 ftp:x:1000:1000:FTP User:/var/ftp:/sbin/nologin
 我们发现ftp用户的家目录在/var/ftp，就是这个/var/ftp的权限不对所致，这个目录的权限是不能打开所有权限的；是您运行了chmod 777 /var/ftp所致；如果没有ftp用户这个家目录，当然您要自己建一个；
url: /archives/492
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - vsftpd

---
**500 OOPS: vsftpd: refusing to run with writable anonymous root**

如果我们已经把vsFTPd服务器启动好了，但登录测试是会出现类似下面的提示；

500 OOPS: vsftpd: refusing to run with writable anonymous root

这表示ftp用户的家目录的权限不对，应该改过才对；

`[root@localhost ~]# more /etc/passwd |grep ftp<br />
ftp:x:1000:1000:FTP User:/var/ftp:/sbin/nologin`

我们发现ftp用户的家目录在/var/ftp，就是这个/var/ftp的权限不对所致，这个目录的权限是不能打开所有权限的；是您运行了chmod 777 /var/ftp所致；如果没有ftp用户这个家目录，当然您要自己建一个；

如下FTP用户的家目录是不能针对所有用户、用户组、其它用户组完全开放；

`[root@localhost ~]# ls -ld /var/ftp<br />
drwxrwxrwx 3 root root 4096 2005-03-23 /var/ftp`

**修正这个错误，应该用下面的办法；**

`[root@localhost ~]# chown root:root /var/ftp<br />
[root@localhost ~]# chmod 755 /var/ftp`

有的弟兄可能会说，那匿名用户的可读、可下载、可上传怎么办呢？这也简单，在/var/ftp下再建一个目录，权限是777的就行了，再改一改vsftpd.conf就OK了；没有什么难的；

vsFTPd出于安全考虑，是不准让ftp用户的家目录的权限是完全没有限制的，您可以去读一下vsFTPd的文档就明白的了；否则也不能称为最安全的FTP服务器了，对不对？