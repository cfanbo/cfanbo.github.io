---
title: '[教程]freebsd下SVN服务器配置'
author: admin
type: post
date: 2010-08-17T07:16:48+00:00
url: /archives/5142
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - svn

---
**注意:**这里主要介绍使用svnserver服务器这种方式,在安装的时候使用的是ipv4,所以最好如果没有必要的话,尽量将ipv6的一些选项给取消.

安装svn服务器软件.由于要通过Web访问SVN所以要加载mod_dav模块，所以在安装apche的时候要添加一些参数:

> #cd /usr/ports/devel/subversion
> #make WITH\_MOD\_DAV\_SVN=yes WITHOUT\_BDB=yes install clean
> #rehash

**下边介绍两种使用方式：**
**第一种方式：使用svnserve服务器**，自己的协议和客户端,在freebsd我在/usr/local/www/apache22/data下用FTP上传了一个blog目录

>  #cd /usr/local/www/apache22/data
>
>
> #svnadmin create myblog
> #svn import blog  -m “init”

上面第三条命令是将blog文件夹里的内容,导入到svn项目中,这种原来的文件就会在在/usr/local/www/apache22/data/myblog/blog这个文件夹以项目管理的方式存在了.和原来的blog里的文件没有任何关系了.有点像复制了一个文件夹似的,参考:.

然后出来提示committed revision 1

> #svnserve -d -r /usr/local/www/apache22/data/

该命令含义为让SVN将此目录作为仓库，并侦听客户端的请求。其中-d的作用为后台模式，而-r的作用为指定服务器的仓库路径。

这时候就可以在他机器上使用TortoiseSVN的Rep0-browers访问 svn://ip[:3690]/myblog 得到当前的副本了
现在只能checkout不能commit。下边配置成可以commit

> #cd /usr/local/www/apache22/data/myblog/conf
> #vi svnserve.conf

修改svnserve.conf文件,去掉password-db=passwd前面的注解,要注意password-db=passwd面不能有空格
保存退出后
vi passwd加入

> #vi /usr/local/www/apache22/data/myblog/conf/passwd

 terry = 123456

现在可以commit了，要求你输入用户名，密码，就是刚配置的

**第二种方式：与apache结合**，这样子可以通过这样的地址来check out了
参考
一样的配置

相关文章: [FreeBSD上安装Trac+SVN+Apache＋SSL](http://www.haohtml.com/server/services/45221.html)