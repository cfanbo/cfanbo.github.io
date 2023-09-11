---
title: 'VSFTP中”Fixing 500 OOPS: vsftpd: refusing to run with writable root inside chroot ()”的解决办法!'
author: admin
type: post
date: 2012-01-11T12:10:28+00:00
url: /archives/12426
IM_contentdowned:
 - 1
categories:
 - 服务器

---
今天在参考以前写的在FreeBSD下配置vsftpd教程的时候.发现以下错误:

After upgrading vsftpd to 2.3.5 you may be getting the following message when trying to log in.

> 500 OOPS: vsftpd: refusing to run with writable root inside chroot ()

This is due to the following update:

> – Add stronger checks for the configuration error of running with a writeable
> root directory inside a chroot(). This may bite people who carelessly turned
> on chroot\_local\_user but such is life.

The problem is that your users root directory is writable(用户根目录可写), which isn’t allowed when using chroot restrictions in the new update. The following command will fix this problem, replace the directory with your users root:

> chmod a-w /home/user

好吧,我们如果启用chroot,必须保证ftp根目录不可写,这样对于ftp根直接为网站根目录的用户不方便,所以建议假如ftp根目录是/home/user,那么网站结构可以这样分,/home/user/log为日志目录,/home/user/web为网站根目录,这样我们就可以去掉/home/user目录的写入权限而不影响网站的正常运行