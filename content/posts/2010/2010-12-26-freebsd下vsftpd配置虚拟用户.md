---
title: Freebsd下Vsftpd配置虚拟用户
author: admin
type: post
date: 2010-12-26T12:06:11+00:00
url: /archives/7229
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - vsftpd

---
注意：教程中的英文词组的首字母应该为小写才对.

很久没有发新篇了，其实是很久没上来看了，前段时间实在太忙，还经常加班，现在终于可以喘口气了。北京的天气，近期真是春光明媚啊，呵呵，是时候外出活动了。上上周末打了词羽毛球，这周末也有计划，嘿嘿，要是身体允许的话周日去爬山吧，香山 或者 邻居推荐的鹫峰。

前段时间主要忙于我的系统下的sebsd的策略设置，但是首先就需要熟悉各种服务本身的配置，有关NAMED的比较简单，但是关于VSFTPD的还比较麻烦，写了份文档，贴在这里，也算留个纪念吧。

FREEBSD 的 VSFTPD设置

**说明**

VSFTPD是一个安全高效的FTP服务软件，得到了广泛的应用。

本地用户经过设置后可以进行ftp访问。而匿名用户的访问经过了转换，在系统中。匿名用户的用户名为ftp, 系统将其属性设置为 根目录 /var/ftp/, 禁止控制台登陆，也就是，该用户只能进行ftp访问。

FreeBSD下vsftpd 的执行程序为 /usr/local/libexec/vsftpd, 一般情况下调用 /usr.local/libexec/vsftpd & 即可启动VSFTPD, 注意，修改 /usr.local/etc/vsftpd.conf文件中的listen要设置为YES.

VSFTPD有两种开机自启动模式: inet模式和standalone模式，推荐使用standalone模式。 两种模式的启动方法依次为：

**1. inet模式**

修改 /etc/inet.conf 修改或添加，使以下行有效：

ftp stream tcp nowait root /usr/sbin/tcpd /usr/local/sbin/vsftpd

并修改 /usr.local/etc/vsftpd.conf文件，使 listen=NO

**2. standalone模式**

修改/etc/inet.conf文件并注释掉vsftpd的启动行。

修改 /usr.local/etc/vsftpd.conf文件，使 listen=YES

在 /usr/local/etc/rc.d/下添加vsftpd 的启动脚本

VSFTPD的基本设置

VSFTPD的配置文件为/usr.local/etc/vsftpd.conf, 有多个选项，下面一一说明：

 Anonymous_enable=yes

 允许匿名登陆, 设置为yes后，ftp用户和anonymous用户都被认为以匿名登陆

 Dirmessage_enable=yes

 切换目录时，显示目录下.message的内容， 可以在vsftpd.conf文件中通过message_file修改文件名

 Local_umask=022

 FTP上本地的文件权限，默认是077， 前面有0认为是8进制，无0认为是十进制

 Connect_from_port_20=yes

 启用FTP数据端口的数据连接

 Xferlog_enable=yes

 激活上传和下传的日志

 Xferlog_std_format=yes

 使用标准的日志格式

 Ftpd_banner=XXXXX

 欢迎信息

 Pam_service_name=vsftpd

 验证方式 *

 Listen=yes

 独立的VSFTPD服务器 *

 Anon_upload_enable=yes

 开放上传权限

 Anon_mkdir_write_enable=yes

 可创建目录的同时可以在此目录中上传文件

 Write_enable=yes

 开放本地用户写的权限

 Anon_other_write_enable=yes

 匿名帐号可以有删除的权限

 Anon_world_readable_only=no

 放开匿名用户浏览权限

 Ascii_upload_enable=yes

 启用上传的ASCII传输方式

 Ascii_download_enable=yes

 启用下载的ASCII传输方式

 Banner_file=/var/vsftpd_banner_file

 用户连接后欢迎信息使用的是此文件中的相关信息

 Idle_session_timeout=600(秒)

 用户会话空闲后10分钟

 Data_connection_timeout=120（秒）

 将数据连接空闲2分钟断

 Accept_timeout=60（秒）

 将客户端空闲1分钟后断

 Connect_timeout=60（秒）

 中断1分钟后又重新连接

 Local_max_rate=50000（bite）

 本地用户传输率50K

 Anon_max_rate=30000（bite）

 匿名用户传输率30K

 Pasv_min_port=50000

 将客户端的数据连接端口改在

 Pasv_max_port=60000

 50000—60000之间

 Max_clients=200

 FTP的最大连接数

 Max_per_ip=4

 每IP的最大连接数

 Listen_port=5555

 从5555端口进行数据连接

 Local_enble=yes

 本地帐户能够登陆

 Write_enable=no

 本地帐户登陆后无权删除和修改文件

 下面这是一组

 Chroot_local_user=yes

 本地所有帐户都只能在自家目录

 Chroot_list_enable=yes

 文件中的名单可以调用

 Chroot_list_file=/任意指定的路径/vsftpd.chroot_list

 前提是chroot_local_user=no

 这又是一组

 Userlist_enable=yes

 在指定的文件中的用户不可以访问

 Userlist_deny=yes

 Userlist_file=/指定的路径/vsftpd.user_list

 又开始单的了

 Banner_fail=/路径/文件名

 连接失败时显示文件中的内容

 Ls_recurse_enable=no

 Async_abor_enable=yes

 one_process_model=yes

 Listen_address=10.2.2.2

 将虚拟服务绑定到某端口

 Guest_enable=yes

 虚拟用户可以登陆

 Guest_username=所设的用户名

 将虚拟用户映射为本地用户

 User_config_dir=/任意指定的路径/为用户策略自己所建的文件夹

 指定不同虚拟用户配置文件的路径

 又是一组

 Chown_uploads=yes

 改变上传文件的所有者为root

 Chown_username=root

 又是一组

 Deny_email_enable=yes

 是否允许禁止匿名用户使用某些邮件地址

 Banned_email_file=//任意指定的路径/xx/

 又是单的

 Pasv_enable=yes

 服务器端用被动模式

 user_config_dir=/任意指定的路径//任意文件目录

 指定虚拟用户存放配置文件的路径


**3. 虚拟用户的设置**

所谓虚拟用户，就是在本地用户中不存在，而又可以远程登陆的ftp用户。一般多采用pam 认证方式。 本节只考虑使用pam_pwdfile.so库文件进行认证。更多的可能采用的是 mysql数据库进行认证。

基本的设置过程如下。

在 vsftp.conf中进行如下设置使 vsftpd 知道采用的是 pam的认证方式。

> guest_enable=YES
> guest_username=virtual # 该用户是系统的本地用户
> pam\_service\_name=vsftpd # 该文件指明pam认证对应的配置文件，默认存储位置/etc/pam.d/

配置 pam认证，复制 /etc/pam.d/ftpd 为 vsftpd

在vsftpd文件前加入以下两句

> auth sufficient /lib/security/pam\_pwdfile.so pwdfile=/etc/vsftpd\_login
> account sufficient pam_permit.so

注： 这样做是为了使本地用户与远程用户都能够登入

pam_pwdfile.so 系统默认没有安装，进入ports/security/ 进行安装，注意安装后的so文件与以上文件路径

生成 /etc/vsftpd_login 文件， 该文件是口令文件，记录了用户名和口令的hash编码

编辑虚拟用户名及口令文件 /root/vsftp.login, 格式如下

用户名：口令

生成以下perl脚本 fileter.pl

###########################

> #! /usr/bin/perl -w
>
> use strict;
>
> \# filter “user:cleartext” lines into “user:md5_crypted”
>
> \# probably requires glibc
>
> while (<>) {
>
> chomp;
>
> (my $user, my $pass) = split /:/, $_, 2;
>
> my $crypt = crypt $pass, ‘$1$’ . gensalt(8);
>
> print “$user:$crypt”;
>
> }
>
> sub gensalt {
>
> my $count = shift;
>
> my @salt = (‘.’, ‘/’, 0 .. 9, ‘A’ .. ‘Z’, ‘a’ .. ‘z’);
>
> my $s;
>
> $s .= $salt[rand @salt] for (1 .. $count);
>
> return $s;
>
> }

shell下运行 perl /root/fileter.pl /root/vsftp.login > /etc/vsftpd\_login删除vsftp.login， 注意vsftpd\_login最好不要带文件类型，pam认证好像忽略类型名

在vsftpd.conf中修改为：

> user\_config\_dir = /usr/loca/etc/virtual/ # 需手工mk

以各用户名为文件名生成各文件，设置用户的访问权限。语法与 vsftpd.conf中相同

重起 vsftp就可以了

转自：