---
title: centos下vsftpd 的虚拟用户安装配置
author: admin
type: post
date: 2011-04-07T02:34:16+00:00
url: /archives/9041
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - vsftpd

---
Vsftp 安装配置

**1.查看是否安装vsftp**

> #rpm –qa|grep vsftpd

如果出现     vsftpd-2.0.5-16.el5_5.1  说明已经安装 vsftp

如果没有安装的话, 需要先安装vsftp

> yum -y install vsftpd

**2.测试 是否安装成功**

（ip 改成自己啊，不要用俺的此次登录为匿名登录 user: anonymous 密码为空 如果成功登录会有下面内容  这说明vsftpd安装成功）

> #service vsftpd start

为 vsftpd 启动 vsftpd：[确定]

> #ftp 192.168.1.107
>
> Connected to192.168.1.107.
>
> 220 (vsFTPd 2.0.5)
>
> 530 Please loginwith USER and PASS.
>
> 530 Please loginwith USER and PASS.
>
> KERBEROS_V4 rejectedas an authentication type
>
> Name(192.168.1.107:root): anonymous
>
> 331 Please specifythe password.
>
> Password:
>
> 230 Loginsuccessful.
>
> Remote system typeis UNIX.
>
> Using binary mode totransfer files.
>
> ftp> bye
>
> 221 Goodbye.

**3.修改配置文件/etc/vsftpd/vsftpd.conf**

> #vi /etc/vsftpd/vsftpd.conf

取消下面内容前面的注释或添加

> anonymous_enable=YES/NO  是否允许匿名用户访问
>
> chroot\_list\_enable=YES　　　限定用户不可以离开主目录
>
> chroot\_list\_file=/etc/vsftpd/chroot_list
>
> local_enable=YES/NO 本地用户是否可以访问 注：如果为NO 则所有虚拟用户都将不能访问原因：虚拟用户访问在主机上其实是以本地用户访问的
>
> pam\_service\_name=vsftpd     pam认证文件名 在/etc/pam.d/vsftpd
>
> guest_enable=YES    启用虚拟用户功能
>
> guest\_username=ftp  指定虚拟用户的宿主用户 –centos 里面已经有内置的ftp用户了（注：此用户在chroot\_list\_file=/etc/vsftpd/chroot\_list文件里所指定的用户）
>
> user\_config\_dir=/etc/vsftpd/vuser_conf 设置虚拟用户个人vsftp的服务配置文件所在目录（此文件后面不能出现空格）
>
> #好像vsftpd用户没有办法修改文件的权限的权限(chmod)，加上这两行就可以了
> virtual_use_local_privs=YES
> chmod_enable=YES

**4.查看是否安装 db4 db4-utils**

> #rpm -qa|grep db4  运行后出现下面内容 说明已经安装可以使用db_load命令（主要是 db4-utils）
>
> db4-devel-4.3.29-10.el5_5.2
>
> db4-4.3.29-10.el5_5.2
>
> db4-devel-4.3.29-10.el5_5.2
>
> db4-4.3.29-10.el5_5.2
>
> db4-tcl-4.3.29-10.el5_5.2
>
> db4-utils-4.3.29-10.el5_5.2

如果没安装则要安装db4-utils

4.1安装db4-utils

> #yum -y install db4-utils

**5. 创建 chroot\_list\_file=/etc/vsftpd/chroot_list文件**

为了安全,将ftp用户锁定在自己的目录里(提示:如果直接用chroot\_local\_user=YES 指令的话,则比较的方便了,此时local\_enable和chroot\_list_file的意义就完全相反了.此类用法见:)

> #vi /etc/vsftpd/chroot\_list (编辑文件把 /etc/vsftpd/vsftpd.conf 中guest\_username的值 ftp 写到文件中,主要是为了安全起见,将ftp用户锁定在自己所属的目录里)

或者直接按下面进行操作

> #touch /etc/vsftpd/chroot_list
>
> \# echo  ftp >> /etc/vsftpd/chroot\_list  (此处ftp 也要是/etc/vsftpd/vsftpd.conf中的guest\_username的值)

**6.创建虚拟用户目录（密码文本）**

> #vi /etc/vsftpd/vftpuser.txtx (奇数行为用户名 ，偶数行为密码)
>
> zz
> aaaaa
>
> ftp1
> zzzzz

**7.生成虚拟用户的db文件**

> #db_load -T -t hash -f /etc/vsftpd/vftpuser.txtx /etc/vsftpd/vftpuser.db

**8.生成虚拟用户的认证文件**

> \# vi /etc/pam.d/vsftpd
>
> #%PAM-1.0
> session    optional    pam_keyinit.so    force revoke
> auth       required     pam_listfile.so item=user sense=denyfile=/etc/vsftpd/ftpusers onerr=succeed
> auth       required     pam_shells.so
> auth       include      system-auth
> account    include     system-auth
> session    include     system-auth
> session    required    pam_loginuid.so

注释掉/etc/pam.d/vsftpd中所有的内容 反正已经不要本地用户的认证了

特别注意操作系统以下区别
32位系统增加以下两句：

> auth      required     pam_userdb.so db=/etc/vsftpd/vftpuser
> account   required     pam_userdb.so db=/etc/vsftpd/vftpuser

64位系统增加以下两句：

> auth   required    /lib64/security/pam_userdb.so db=/etc/vsftpd/vftpuser
> account required    /lib64/security/pam_userdb.sodb=/etc/vsftpd/vftpuser

注：db=/etc/vsftpd/vftpuser 中的vftpuser 是你生成的虚拟用户的db文件

查看系统是多少位的命令

> \# getconf LONG_BIT
>
> 64 （64|32）

**9.创建每个虚拟用户自己的配置文件，配置文件的路径是/etc/vsftpd/vsftpd.conf中的**

> user\_config\_dir=/etc/vsftpd/vuser_conf #路径

在 /etc/vsftpd/vuser_conf/下面创建以用户名为名称的文件（/etc/vsftpd/vftpuser.txtx 文件内容的奇数行）

> \# cat /etc/vsftpd/vftpuser.txtx
> zz
> aaaaa
> ftp1
> zzzzz

> \# mkdir vuser_conf
> \# vi /etc/vsftpd/vuser_conf/zz

内容如下

> local_root=/var/www(虚拟用户的根目录根据实际修改)
> write_enable=YES （可写）
> download_enable=YES
> anon\_world\_readable_only=NO
> anon\_upload\_enable=YES
> anon\_mkdir\_write_enable=YES
> anon\_other\_write_enable=YES
> local_umask=022

**10.给文夹权限(否则不能上传 权限可自定 本人给的是 777,如果指定的为其它的目录,则目录所有者应该为ftp:ftp这个用户,如果上传的为网站程序内容之类的文件的话,则需要变通一下,参考:)**

> #chown -R ftp:ftp /var/www/
> #chmod -R 755 /var/www/

**11.重启vsftpd**

> \# service vsftpd restart

到此安装配置完成 如果出现连接被 同位体重置 或其它错误 请查看SELinux的当前模式

**12.登录测试**

> \# ftp192.168.1.107
>
> Connected to 192.168.1.107.
>
> 220 (vsFTPd 2.0.5)
>
> 530 Please login with USER and PASS.
>
> 530 Please login with USER and PASS.
>
> KERBEROS_V4 rejected as anauthentication type
>
> Name (192.168.1.107:root): zz
>
> 331 Please specify the password.
>
> Password:
>
> 500 OOPS: cannot changedirectory:/var/www
>
> Login failed.
>
> ftp>

查看方法

> \# getenforce
>
> Enforcing   如果出现（Enforcing ）
>
> 关闭方法：
> #setenforce 0       （0|1  开|关）



再次测试 登录成功

> \# chmod 777/var/www/
>
> \# ftp192.168.1.107
>
> Connected to 192.168.1.107.
>
> 220 (vsFTPd 2.0.5)
>
> 530 Please login with USER and PASS.
>
> 530 Please login with USER and PASS.
>
> KERBEROS_V4 rejected as anauthentication type
>
> Name (192.168.1.107:root): zz
>
> 331 Please specify the password.
>
> Password:
>
> 230 Login successful.
>
> Remote system type is UNIX.
>
> Using binary mode to transfer files.
>
> ftp>

对于创建vsftpd虚拟用户上传网站程序造成的不同用户和组权限的问题,可以参考FreeBSD下的vsftpd解决方案:,这里是将ftp用户归属到www用户组里了.基本上通过ftp用户上传的程序,可以以网页的形式打开的,只要把目录权限设置为770就可以了,不过有点小问题需要注意,在程序里如果创建的目录和文件的话,一定要把权限设置为770,否则创建的目录和文件,没有办法通过ftp直接删除,只能通过web的方法才能解决的.