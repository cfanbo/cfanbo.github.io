---
title: Linux下配置vsftpd若干问题集锦
author: admin
type: post
date: 2010-12-27T10:50:16+00:00
url: /archives/7299
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - debian
 - vsftpd

---
debian上配置vsftpd若干问题集锦

**1.debian上如何安装vsftpd**
很简单apt-get install vsftpd

**2.vsftpd的去除匿名用户登录问题**
vi /etc/vsftpd.conf
anonymous_enable=YES
修改为
anonymous_enable=NO

**3.如何更改vsftpd的默认端口**
vi /etc/vsftpd.conf
新增一行
listen_port=2010

**4.如何允许本地用户登录**
#local_enable=YES前面的#去掉

**5.如何允许用户可以上传文件**
#write_enable=YES前面的#去掉

**6.如何添加一个新用户**
由于我是用本地用于登录的模式，所以确定你的local_enable=YES已经开启，再做下面的工作
首先添加一个用户组
groupadd ftpgroup
然后添加用户
useradd blogguy_cn –g ftpgroup –d /home/blogguy.cn –s /bin/bash
passwd blogguy_cn
输入新密码，即可生效

**7.如何限制新添加的用户不能使用bash登录**
首先检查的你的shells文件
vi /etc/shells
看看有没这两行
/sbin/nologin
/bin/false
没有的话加上，然后更新用户的bash
usermod -s /sbin/nologin blogguy\_cn或者usermod -s /bin/false blogguy\_cn
查看你的passwd文件是否生效
cat /etc/passwd

**8.如何锁定用户在自己的主目录**
有两个方法，我分别说
第一个方法：一刀切，把所有的用户都限制在自己的主目录
chroot_local_user=YES  把前面的注释去掉即可
好像不生效
第二个方法：限制部分用户在自己的主目录
首先

> vi /etc/vsftpd.chroot_list

把你要限制的用户一行一个的添加进去
然后开启vsftpd.conf的

> #chroot\_list\_enable=YES
> #chroot\_list\_file=/etc/vsftpd.chroot_list

把#号去掉即可

**9.如何只允许部分系统用户登录ftp**
首先新建一个允许登录的用户文件
vi /etc/vsftpd.user_list
输入允许登陆的用户名blogguy_cn，一行一个
然后vi /etc/vsftpd.conf
添加以下几行

> userlist_enable=YES
> userlist_deny=NO
> userlist\_file=/etc/vsftpd.user\_list

解释：userlist_file是用文件地址
userlist\_enable=YES表示是否开启用户用户列表也能，如果设置为YES就会读取userlist\_file文件内容，但只此刻什么事都都不做。
userlist\_deny表示允许或者拒绝，若果设置为userlist\_deny=NO表示若设为NO ， 则只有在/etc/vsftpd.user\_list 中的使用者才能登入，而且此项功能可以在询问密码前就出现错误讯息，而不需要检验密码的程序。 若果设置为userlist\_deny=YES，表示在userlist的用户不能登录

**10.如何上匿名访问、上传，并支持下载和执行？**

在默认的情况下，vsftp是不支持匿名用户的访问的，所以我们要自己打开相应的选项。现在我针对这个问题，我们要打开如下的选项。

> anonymous_enable=YES 注：允许匿名访问
> anon\_upload\_enable=YES 注：允许上传
> anon\_mkdir\_write_enable=YES 注：允许建立相应的目录
> anon_umask=022 把上传到FTP的文件或者目录改变权限

当然打开这些选项还是不行的，我们还要让匿名写入文件的上一级目录有写入权，以我所做的FTP为例，我所做的FTP的匿名访问的目录是/var /ftp，在vsFTPd中，/var/ftp这个目录是不能让匿名用户有写入权限的，这是为了安全考虑，所以我们必须自己在/var/ftp目录中建一 个目录，让这个目录有写入权。

比如：我在/var/ftp目录建一个upload目录，然后把它的权限设置成777，这样匿名用户就能写入了。

> #mkdir /var/ftp/upload
> #chmod 777 /var/ftp/upload

改了一系列的文件，不要忘记重启vsFTPd服务器

**11.如何解决vsftp上传的文件与apache或者nginx的权限问题**
这个问题不少人问，但是回答的很少，这里我给一个的回答
vi /etc/vsftpd.conf
找到#local_umask=022把前面的#去掉
重启服务器试试
具体各项代表什么意思，抄录一段台湾人的
設定值 結果 代表的使用權限
666 002 664 rw-rw-r–
666 022 644 rw-r–r–
666 037 640 rw-r—–
666 077 600 rw——-
目錄權限的最大值
設定值 結果 代表的使用權限
777 002 775 rwxrwxr-x
777 022 755 rwxr-xr-x
777 037 740 rwxr—–
777 077 700 rwx——