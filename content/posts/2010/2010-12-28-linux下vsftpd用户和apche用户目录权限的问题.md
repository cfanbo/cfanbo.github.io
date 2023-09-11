---
title: linux下vsftpd用户和apche用户目录权限的问题
author: admin
type: post
date: 2010-12-28T14:13:15+00:00
url: /archives/7347
IM_contentdowned:
 - 1
categories:
 - 服务器

---
比如我的网站的目录在/var/www/demo下，其中网站根目录下有个upload文件夹是专门用来上传图片的。

所以我把这个目录的权限设置为了 777 ，然后通过php程序自动在upload目录下建立了一个文件夹090602，并在090602下通过程序上传一个1.jpg到这个目录下，这样出现了问题一：我通过客户端的flashfxp连接上去之后不能删除090602这个目录及其下的1.jpg，原因是这个090602和1.jpg的所有者是apache系统下的daemon组的daemon 。

问题二：我现在通过flashfxp以newuser(它是属于我新建的一个组flashfxp)登录vsftpd并在网站的upload目录下建立一个090603目录，但这样到了09年6月3号的时候php程序却不能在090603这个目录下上传文件了 。

请问有什么好的方法让upload目录下的所有目录及文件同时属于flashfxp组的newuser用户和apache系统下的daemon组的daemon用户呢？或者大家有什么更好的方法呢？

> 呵呵，解决了,方法如下：
> 把 newuser 和 daemon 这两个用户都添加到daemon组，然后执行命令：chmod -R g+rwx /var/www/demo

这里有详细的文档：