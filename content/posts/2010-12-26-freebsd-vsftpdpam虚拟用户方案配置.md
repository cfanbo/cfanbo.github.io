---
title: '[教程]FreeBSD vsftpd+pam虚拟用户方案配置'
author: admin
type: post
date: 2010-12-26T10:28:07+00:00
url: /archives/7213
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - 虚拟用户
 - pam
 - vsftpd

---
**1. vsftpd安装**

> ****cd /usr/ports/ftp/vsftpd
> make install clean

**2. vsftpd的配置文件与启动文件**

> ****(1)配置文件的位置 /usr/local/etc/vsftpd.conf
> (2)启动文件的位置 /usr/local/libexec/vsftpd

**3. vsftpd虚拟用户的配置**

> vi /usr/local/etc/vsftpd.conf
>
> anonymous_enable=NO
> local_enable=YES
> write_enable=YES
> local_umask=022
>
> anon\_upload\_enable=NO
> anon\_mkdir\_write_enable=NO
>
> #限制本地用户在自己的目录里，这里将chroot_list_enable=YES 和chroot_list_file=/任意路径/vsftpd.chroot_list 注释掉(切记:以后添加新ftp账户的时候,需要在此文件里也添加一行,来对用户进行锁定在自己的目录里,否则是非常的危险的)
> chroot_local_user=YES

#chroot_list_enable=YES

#chroot_list_file=/etc/vsftpd.chroot_list
>
> listen=YES
> secure\_chroot\_dir=/usr/local/share/vsftpd/empty
> background=YES
>
> —————————————————————–
>
> listen_port=5123     #为了安全,这里修改ftp的端口号
> pam\_service\_name=vsftpd
> **user_config_dir=/usr/local/etc/vsftpd/**
>
> #好像vsftpd用户没有办法修改文件的权限的权限(chmod)，加上这两行就可以了
> virtual_use_local_privs=YES
> chmod_enable=YES
>
> #用户虚拟用户

 guest_enable=YES

guest_username=virtual

**4. 创建虚拟用户目录与用户的配置文件**

 ****用哪个系统用户做为虚拟用户的用户，权限就是谁的，例如用www，作为虚拟用户映射的用户。权限就应该为www。通常该方法用于网站的代码的上传。

一.添加ftp对应的本地系统用户virtual,用户组为www

> #pw useradd virtual –g www–d /home/virtual –s /sbin/nologin
> #mkdir /home/virtual
> #passwd virtual

二.创建ftp虚拟用户配置文件

> **# mkdir /usr/local/etc/vsftpd**
> #注意这里的路径是user\_config\_dir 的路径

> #cd /usr/local/etc/vsftpd
> \# vi abc
> guest_enable=YES
> guest_username=virtual
> local_root=/data/htdocs/www
> anon\_world\_readable_only=no
> anon\_upload\_enable=yes
> anon\_mkdir\_write_enable=yes
> anon\_other\_write_enable=yes
> anon_umask=022

用户virtual为本地www组的本地用户，意思是将虚拟用户映射为本地用户(这里可能有问题,virtual用户默认的sh为nologin的,主目录为/home/virtual,并非/nonexistent,所以如果用ftp连接的时候发现直接就提示 “500 OOPS: cannot change directory:/nonexistent“，请检查本地用户的主目录)。另外虚拟用户的目录一定让相对应的本地用户对目录有相应的访问操作权限。

> #mkdir -p /data/htdocs/www
> #chown -R virtual:www /data/htdocs/www
> \# chmod -R 775 /data/htdocs/www

其中上面的一些选项，可以在vsftpd.conf里进行全局配置的，这样再添加用户的时候就方便多了。这里有些配置是重复的.但不受影响的.

**5. 安装vsftpd密码认证模块**

> ****cd /usr/ports/security/pam_pwdfile
> make install clean

用于生成/usr/local/lib/pam_pwdfile.so文件
注:pam_pwdfile.so, 有关PAM认证模块简介见:

**6. 创建vsftpd认证模块**
创建认证模块.记得清除默认配置文件里的auth和account内容或者注释掉。

> \# cp /etc/pam.d/ftpd /etc/pam.d/vsftpd
> #vi /etc/pam.d/vsftpd
> auth          required  /usr/local/lib/pam_pwdfile.so  pwdfile=/usr/local/etc/login
> account    required   pam_permit.so

请注意下面的required认证处理的方式,一定要为required,不能为其它的,如sufficient ,有关他们的区别,请参考:

**7. 创建用户密码**
可以用以下代码来实现用户密码的加密，用于ftp的用户验证。以下内容由于wordpress程序问题,可能显示或者复制的时候有误,请上传一张图.

[![](http://blog.haohtml.com/wp-content/uploads/2010/12/add_ftp_user.png)][1]

复制下面代码:

>

```
vi /usr/local/etc/add_ftp_user.pl
代码如下:
#! /usr/bin/perl -w
#filename: md5pwd.pl
use strict;
#
print "#example: user:passwd\n";
while (<STDIN>) {
    exit if ($_ =~/^\n/);
    chomp;
    (my $user, my $pass) = split /:/, $_, 2;
    my $crypt = crypt $pass, '$1$' . gensalt(8);
    print "$user:$crypt\n";
}
sub gensalt {
    my $count = shift;
    my @salt = ('.', '/', 0 .. 9, 'A' .. 'Z', 'a' .. 'z');
    my $s;
    $s .= $salt[rand @salt] for (1 .. $count);
    return $s;
}
```

> \# chmod +x /usr/local/etc/add\_ftp\_user.pl

> \# /usr/local/etc/add\_ftp\_user.pl
> #example: user:passwd
> abc:abc
> abc:$1$gLAEihTV$jQnPZDk4C8TZSrc.L7gLm/

说明:然后把上面的用户名与密码文件复制到下面的login文件中(可以一次添加多个用户及密码,按ctrl+c结束)

> \# vi /usr/local/etc/login
> abc:$1$gLAEihTV$jQnPZDk4C8TZSrc.L7gLm/

**9. 启动服务**
启动:

> /usr/local/libexec/vsftpd
> 或者
> /usr/local/etc/rc.d/vsftpd start
>
> Usage: /usr/local/etc/rc.d/vsftpd \[fast|force|one\](start|stop|restart|rcvar|reload|status|poll)
>
>

**关闭**:直接kill掉

> #killall vsftpd

这里将vsftpd作为服务启用,写到入/etc/rc.conf

> echo ‘vsftpd_enable=”YES”‘ >> /etc/rc.conf

**10. 测试vsftpd的登录情况**

略，和下面的一样.

================================

**11.添加新用户**

下面我们来添加一个新用户

> #/usr/local/etc/add\_ftp\_user.pl
> #example:user:password
> haohtml:com
> haohtml:$1$b5cRPRVT$MODPUh9H0F1JWaioqbrXB.

> echo ‘haohtml:$1$b5cRPRVT$MODPUh9H0F1JWaioqbrXB.’ >> /usr/local/etc/login

添加配置文件:

> vi /usr/local/etc/vsftpd/haohtml

将下面内容添加到文件里(这里为了方便，将guest_enable=YES和guest_username=apache放在了全局配置文件vsftpd.conf里了)

>

> local_root=/data/htdocs/haohtml
>

>
>

> anon_world_readable_only=no
>

>
>

> anon_upload_enable=yes
>

>
>

> anon_mkdir_write_enable=yes
>

>
>

> anon_other_write_enable=yes
>

>
>

> anon_umask=022
>

创建ftp目录

> #mkdir /data/htdocs/haohtml
> #chown -R virtual:www /data/htdocs/haohtml
> #chmod -R 775  /data/htdocs/haohtml

**重启vsfptd 生效**

> #/usr/local/libexec/vsftpd

ftp测试：

[![FreeBSD vsftpd+pam虚拟用户方案配置](http://blog.haohtml.com/wp-content/uploads/2010/12/freebsd_vsftpd_virtual.jpg)][2]

有关配置参数请参考:

注意这里我们使用了chroot的功能,新版本的vsftpd软件(发现2.3.5版本有此问题.但2.3.2无此问题),可能会提示一些”Fixing 500 OOPS: vsftpd: refusing to run with writable root inside chroot “错误,解决办法只需要将ftp虚拟用户对应的根目录的写权限取消即可.参考:

> chmod a-w /data/htdocs/haohtml**相关教程:**[http://linux.chinaunix.net/bbs/thread-1000544-1-1.html](http://linux.chinaunix.net/bbs/thread-1000544-1-1.html)[http://linux.chinaunix.net/bbs/viewthread.php?tid=484844](http://linux.chinaunix.net/bbs/viewthread.php?tid=484844)[http://blog.163.com/koumm@126/blog/static/9540383720102162529276/](http://blog.163.com/koumm@126/blog/static/9540383720102162529276/)**对于Linux(Centos)下配置vsftpd的方法请参考:**

 [1]: http://blog.haohtml.com/wp-content/uploads/2010/12/add_ftp_user.png
 [2]: http://blog.haohtml.com/wp-content/uploads/2010/12/freebsd_vsftpd_virtual.jpg