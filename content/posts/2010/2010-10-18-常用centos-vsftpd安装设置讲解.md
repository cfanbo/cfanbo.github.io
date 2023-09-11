---
title: '[教程]常用CentOS vsftpd安装设置讲解'
author: admin
type: post
date: 2010-10-18T02:22:31+00:00
url: /archives/6187
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - vsftpd

---
CentOS vsftpd还是比较常用的，于是我研究了一下CentOS vsftpd，在这里拿出来和大家分享一下，希望对大家有用。这里讲解介绍centos vsftpd的设置。CentOS Linux与RHEL产品有着严格的版本对应关系，例如使用RHEL 4源代码重新编译发布的是CentOS Linux 4.0，与RHEL 5对应的是CentOS Linux 5.0。

本地用户经过设置后可以进行ftp访问。而匿名用户的访问经过了转换，在系统中。匿名用户的用户名为ftp, 系统将其属性设置为 根目录 /var/ftp/, 禁止控制台登陆，也就是，该用户只能进行ftp访问。CentOS vsftpd 的执行程序为 /etc/vsftpd，修改 /etc/vsftpd/vsftpd.conf文件中的listen要设置为YES.

CentOS vsftpd有两种开机自启动模式: inet模式和standalone模式，推荐使用standalone模式。
在CentOS中已集成了CentOS vsftpd软件。CentOS vsftpd是一个安全高效的FTP服务软件，得到了广泛的应用。

**一、CentOS vsftpd安装**

在服务中查看是否已安装VSFTPD服务。如没有，下载并安装：

> yum install vsftpd

**二、设置CentOS vsftpd自启动**

> ****chkconfig –level 35 vsftpd on

**三、CentOS vsftpd配置**

1. 打开 /etc/vsftpd/vsftpd.conf文件。将anonymous_enable=YES，改为anonymous_enable=NO
2. 打开 /etc/vsftpd/vsftpd.conf文件。添加user_config_dir=/etc/vsftpd/virtual，并建立virtual目录。在此目录中建立以ftp用户名为文件名的文件，并写入以下基本配置：

> mkdir /data/ftp/virtual

>

> local_root=/data/ftp/test
>

>
>

> write_enable=YES
>

>
>

> anon_umask=022
>

>
>

> anon_world_readable_only=NO
>

>
>

> anon_upload_enable=YES
>

>
>

> anon_mkdir_write_enable=YES
>

>
>

> anon_other_write_enable=YES
>

local_root=[目录]，这个目录即是FTP连接时的主目录。

3. 限定用户只在自己目录：修改vsftpd.conf文件，取消注释：

> chroot\_list\_enable=YES
> chroot\_list\_file=/etc/vsftpd/chroot_list

在/etc/vsftpd/目录下添加文件chroot_list，加入作为FTP用户的本地用户名。

4.不显示所有隐藏文件,在vsftpd.conf文件末尾添加下行

> hide_file=.*

5 解决用户无法进入目录问题：
打开终端，输入：

> setsebool -P ftpd\_disable\_trans 1

然后重启FTP服务：

> service vsftpd restart

**四、权限：**

假设是/var/www/html
这个目录的权限应该是770，owner是test，group是ftp

> chmod 770 /var/www/html
> chown test:ftp /var/www/html

**五、添加用户**

添加一个ftp虚拟用户:test,组为ftp,ftp目录为/data/ftp/test,并禁止系统登录:

> /usr/sbin/adduser -d /data/ftp/test -g ftp -s /sbin/nologin test
> passwd test //设置ftp用户密码

再按照上面第三步的操作进行设置即可.

相关文章:[解决vsftpd虚拟用户没有chmod权限的问题](http://blog.haohtml.com/archives/7852)虚拟用户设置: