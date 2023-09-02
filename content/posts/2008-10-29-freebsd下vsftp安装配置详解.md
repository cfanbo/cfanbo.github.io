---
title: '[教程]FreeBSD下vsftp安装配置详解(ports方式)'
author: admin
type: post
date: 2008-10-29T12:22:34+00:00
url: /archives/484
IM_data:
 - 'a:1:{s:38:"/wp-content/uploads/2009/01/vsftp1.jpg";s:38:"/wp-content/uploads/2009/01/vsftp1.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - vsftpd

---
FreeBSD功能强大,ftp服务器只是它其中的很基础的一种服务,但是作为日常的服务器运作ftp服务却是必不可少,本篇是本人自己在学习FreeBSD的服务器设置过程中的一些积累,因为自己也曾是由菜鸟入门,走了不少弯路,现在把自己的一些经验总结出来,供大家参考,希望对新人能有所帮助,不足之处还请大家多多指点.

**1、安装**

通过ports安装，这个方式比较简单。

> \# cd /usr/ports/ftp/vsftpd
> \# make install

[![](/wp-content/uploads/2009/01/vsftp1.jpg)][1][
][2]

> 安装过程中会弹出一个对话框架,选中第一个选项,我以前没有选中,结果安装完以后,在/usr/local/etc/rc.d/目录里没有vsftpd这个命令,导致启动的时候出现以下错误信息:
> **”500 OOPS: vsftpd: cannot open config file:start”**

**2、配置
**

/usr/local/etc/vsftpd.conf文件一般按以下配置就差不多了:

> **anonymous_enable=NO**
>
> **local_enable=YES**
>
> **write_enable=YES**
>
> **local_umask=022**
>
> **chroot_list_enable=YES (开启锁定用户目录)**
>
> **listen＝YES**
>
> ****background=YES****

(1)编辑/usr/local/etc/vsftpd.conf

\# ee /usr/local/etc/vsftpd.conf

Anonymous_enable=NO (禁止匿名登陆)

Local_enable=YES (允许本地用户登陆)

Local_umask=022 (FTP上本地的文件权限755，默认是077)

Connect\_form\_port_20=yes (启用FTP数据端口的数据连接)

Xferlog_enable=yes (激活上传和下传的日志)

Xferlog\_std\_format=yes (使用标准的日志格式)

Idle\_session\_timeout=120(秒) (用户会话空闲后2分钟)

Data\_connection\_timeout=300(秒) (将数据连接空闲5分钟断)

Ascii\_upload\_enable=YES (起用ASCII方式上传)

Ascii\_download\_enable=YES帮带(起用ASCII方式下载)

Ftpd_banner=Welcome to blah FTP service. (FTP服务器登陆欢迎信息)

Chroot_list_enable=YES (开启锁定用户目录)

Chroot_list_file=/任意路径/vsftpd.chroot_list (开启锁定用户目录后，凡在这个文件里出现的用户名都不起使用,格式为每个用户一行,要与Chroot)\_list\_enable=YES 配对使用)

注：如果想把本地的任何用户都锁定在自己的目录中的话，把上面两行(Chroot\_list\_enable 和 Chroot\_list\_file)注释掉，再增加这一样

Chroot_local_user＝YES

保存退出

(2)编辑/etc/inetd.conf

\# ee /etc/inetd.conf

增加这一行并去掉前面的注释(#号)

#ftp   stream tcp    nowait root    /usr/local/libexec/vsftpd     vsftpd

保存退出

**(3)编辑/etc/rc.conf**

\# ee /etc/rc.conf

增加下面内容：

inetd_enable=”YES”

注：以上是以inetd的方式启动vsftp的，我们也可以以独立进程的方式启动vsftp，具体如下：

a、注释掉inetd里面的vsftpd这一行。

b、在vsftpd.conf文件里增加

> **listen＝YES
> background=YES
>**

c、启用ftp服务:

> **#/usr/local/etc/rc.d/vsftpd restart**
>
> 或者
>
> **#/usr/local/libexec/vsftpd &**

d、想要让vsftp随系统启动，这里有两种方法:
法一：在/etc/rc.conf文件里添加

> **vsftpd_enable=”YES”.**

****法二：在/usr/local/etc/rc.d/目录里增加一个sh脚本：

> \# vi vsftpd_start.sh
>
> \# ! /bin/sh
>
> /usr/local/libexec/vsftpd &

保存退出,再chmod 755 vsftpd_start.sh 。

**(4)添加用户**

> \# pw groupadd vsftpd –g 1001
>
> \# pw useradd test –g 1001–d /home/test –s /sbin/nologin
>
> \# mkdir /home/test
>
> \# passwd test               设密码

> Changing local password for test
>
> New Password:
>
> Retype New Password:
>
> #

上面的 -d /home/test 为用户的默认默认,这里改为ftp目录的路径,这样用户直接登录后的位置就是ftp的位置了.

在**vsftpd.chroot_list**文件里增加 test 一行，把test用户锁在其自家目录下。

> #touch /etc/vsftpd.chroot_list
> #echo ‘test’  >> /etc/vsftpd.chroot_list

> \# killall －HUP inetd，(如果是独立进程则执行上面写的那个脚本即可)测试一下：
>
> \# ftp localhost

如果成功会提示你输入用户名和密码

如果不成功，请查看一下你上面的配置

一般不成功是由于权限引起的问题,如无法上传文件,此时可以用ls -al查看一下FTP文件夹的详细权限,一般设置文件的权限为
**-rw- r– r- –**

```
就可以了.
```

看文件夹所有者是不是ftp用户组里的用户,还是用户组是不是正确的,如果说不正确可以用chmod命令修改一下所有者,如修改文件haohtmlcom文件夹的所有都是vsftpd用户组里的test用户,执行**#chmod -R 701 /usr/local/www/haohtmlcom**命令即可,这样此文件夹只能允许vsftpd组里的test用户访问,组中的其它用户无法进行访问(700),当然也可以设置其它用户组里用户,这种情况一般用于为一个站点开设两个ftp用户时使用,只需要执行**#chmod -R 771 /usr/local/www/haohtmlcom**即可.上面的**-R**参数是为了所权限应用到子目录,不然只能当前目录起作用的,无法达到用户的使用要求.
**
#chown -R test:vsftpd /****usr/local/www/haohtmlcom**
如果还不行请参考: [http://blog.haohtml.com/index.php/archives/2682](/index.php/archives/2682)

**启动vsftp服务**
**\# /usr/local/etc/rc.d/vsftpd restart**

**(5)用户功能权限配置**

以下是一些用户的配置：

Anonymous_enable=yes (允许匿名登陆)

Dirmessage_enable=yes (切换目录时，显示目录下.message的内容)

Local_umask=022 (FTP上本地的文件权限，默认是077)

Connect\_form\_port_20=yes (启用FTP数据端口的数据连接)

Xferlog_enable=yes (激活上传和下传的日志)

Xferlog\_std\_format=yes (使用标准的日志格式)

Ftpd_banner=XXXXX (欢迎信息)

Pam\_service\_name=vsftpd (验证方式)

Listen=yes (独立的VSFTPD服务器)

Anon\_upload\_enable=yes (开放上传权限)

Anon\_mkdir\_write_enable=yes (可创建目录的同时可以在此目录中上传文件)

Write_enable=yes (开放本地用户写的权限)

Anon\_other\_write_enable=yes (匿名帐号可以有删除的权限)

Anon\_world\_readable_only=no (放开匿名用户浏览权限)

Idle\_session\_timeout=600(秒) (用户会话空闲后10分钟)

Data\_connection\_timeout=120(秒) (将数据连接空闲2分钟断)

Accept_timeout=60(秒) (将客户端空闲1分钟后断)

**注意：**此方法好像一般不用的，为了安全一般在vsftpd下创建虚拟用户的方法的，详见： [FreeBSD vsftpd+pam虚拟用户方案配置](../index.php/archives/7213)

 [1]: /wp-content/uploads/2009/01/vsftp1.jpg
 [2]: /wp-content/uploads/2009/01/vsftp.jpg