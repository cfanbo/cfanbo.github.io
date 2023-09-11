---
title: centos下安装vsfptd架设ftp服务器
author: admin
type: post
date: 2010-10-16T06:26:53+00:00
url: /archives/6159
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - vsfptd

---
**1.安装vsftp**

在这里，我们架设的是虚拟用户，所谓虚拟用户就是没有使用真实的帐户，只是通过某种手段达到映射帐户和设置权限的目的。

> yum install vsftpd
> touch /var/log/vsftpd.log #创建vsftp的日志文件

在CentOS中，这样就可以完成了一个简单的匿名FTP的搭建。你可以通过访问ftp://yourip来进行，不过这个FTP没有任何权限。
**2.启动/重启/关闭vsftpd服务器**

> ****[root@localhost ftp]# /sbin/service vsftpd restart
> Shutting down vsftpd: [ OK ]
> Starting vsftpd for vsftpd: [ OK ]

OK表示重启成功了.
启动和关闭分别把restart改为start/stop即可.


如果是源码安装的,到安装文件夹下找到start.sh和shutdown.sh文件,执行它们就可以了.
**3.与vsftpd服务器有关的文件和文件夹**
vsftpd服务器的配置文件的是: /etc/vsftpd/vsftpd.conf
vsftpd服务器的根目录,即FTP服务器的主目录:/var/ftp/pub
如果你想修改服务器目录的路径,那么你只要修改/var/ftp到别处就行了

**4.添加FTP本地用户（即虚拟用户,简单方案）**
有的FTP服务器需要用户名和密码才能登录,就是因为设置了FTP用户和权限.
FTP用户一般是不能登录系统的,只能进入FTP服务器自己的目录中,这是为了安全.
这样的用户就叫做虚拟用户了.实际上并不是真正的虚拟用户,只是不能登录SHELL了而已,没权限登录系统.

> /usr/sbin/adduser -d /opt/test_ftp -g ftp -s /sbin/nologin test

这个命令的意思是:
使用命令(adduser)添加test用户,不能登录系统(-s /sbin/nologin),自己的文件夹在(-d /opt/test_ftp)),属于组ftp(-g ftp)
然后你需要为它设置ftp登录密码 passwd test

> passwd test
> Changing password for user test.
> New UNIX password:
> Changing password for user test.New UNIX password:

注意：这里的密码要求为字母和数字的组合才可以，如果不符合密码验证机制的话就修改不成功，会有各种报错，“BAD PASSWORD: it’s WAY too short”，这是报密码太短，不符合/etc/login.defs的设置，“BAD PASSWORD: it is based on your username”，这是密码与帐号不能同名，这是不符合/etc/pam.d/passwd的设置。“BAD PASSWORD: it is based on a dictionary word”这是因为出现了字典里的字符串，如果你英文与数字组合使用，就不会报错。

———————————————————————————————————-
1）我们在/etc/vsftpd/vsftpd.conf中做如下CentOS FTP服务配置：（复杂方案）
anonymous_enable=NO 设定不允许匿名访问
local_enable=YES 设定本地用户可以访问。注：如使用虚拟宿主用户，在该项目设定为NO的情况下所有虚拟用户将无法访问。
chroot\_list\_enable=YES 使用户不能离开主目录
xferlog_file=/var/log/vsftpd.log 设定vsftpd的服务日志保存路径。注意，该文件默认不存在。必须要手动touch出来
ascii\_upload\_enable=YES
ascii\_download\_enable=YES 设定支持ASCII模式的上传和下载功能。
pam\_service\_name=vsftpd PAM认证文件名。PAM将根据/etc/pam.d/vsftpd进行认证
以下这些是关于Vsftpd虚拟用户支持的重要CentOS FTP服务配置项目。
默认vsftpd.conf中不包含这些设定项目，需要自己手动添加CentOS FTP服务配置。

> guest_enable=YES 设定启用虚拟用户功能。
> guest_username=ftp 指定虚拟用户的宿主用户。-CentOS中已经有内置的ftp用户了
> user\_config\_dir=/etc/vsftpd/vuser_conf 设定虚拟用户个人vsftp的CentOS FTP服务文件存放路径。

存放虚拟用户个性的CentOS FTP服务文件(配置文件名=虚拟用户名)
2）创建chroot list，将用户ftp加入其中：

> touch /etc/vsftpd/chroot_list
> echo test >> /etc/vsftpd/chroot_list

3）进行认证（可以不认证）：
首先，安装Berkeley DB工具，很多人找不到db_load的问题就是没有安装这个包。

> yum install db4 db4-utils

然后，创建用户密码文本/etc/vsftpd/vuser_passwd.txt ，注意奇行是用户名，偶行是密码
ftpuser1
ftppass1
ftpuser2
ftppass2
接着，.生成虚拟用户认证的db文件

> db\_load -T -t hash -f /etc/vsftpd/vuser\_passwd.txt /etc/vsftpd/vuser_passwd.db

随后，编辑认证文件/etc/pam.d/vsftpd，全部注释掉原来语句
再增加以下两句

> auth required pam\_userdb.so db=/etc/vsftpd/vuser\_passwd
> account required pam\_userdb.so db=/etc/vsftpd/vuser\_passwd

最后，创建虚拟用户个性CentOS FTP服务文件

> mkdir /etc/vsftpd/vuser_conf/
> vi /etc/vsftpd/vuser_conf/ftpuser1

内容如下：

> local_root=/opt/var/ftp1 虚拟用户的根目录(根据实际修改)
> write_enable=YES 可写
> anon_umask=022 掩码
> anon\_world\_readable_only=NO
> anon\_upload\_enable=YES
> anon\_mkdir\_write_enable=YES
> anon\_other\_write_enable=YES

————————————————————————————————————————-

**5、常见错误：**

安装完以后，可能发现连接ftp服务器，一般是由于SELinux的问题，原因如下：

他的系统是CentOS，是RH派系的。我把vsftpd安装配置好了，以为大功告成，但客户端访问提示如下错误：
 **500 OOPS: cannot change directory:/home/ftp**
原因是他的CentOS系统安装了SELinux，因为默认下是没有开启FTP的支持，所以访问时都被阻止了。
 //查看SELinux设置

> ****\# getsebool -a|grep ftp
> ftpd\_disable\_trans –> off
> ftp\_home\_dir–>off

//使用setsebool命令开启

> \# setsebool ftpd\_disable\_trans 1
> \# setsebool ftp\_home\_dir 1

由于操作系统一旦重启后，这种设置需要重新设置，这里使用-P参数实现.

//setsebool使用-P参数，无需每次开机都输入这个命令

> \# setsebool -P ftpd\_disable\_trans 1
> \# setsebool -P ftp\_home\_dir 1

//查看当前状态是否是on的状态

> \# getsebool -a|grep ftp
> ftpd\_disable\_trans –> on
> ftp\_home\_dir–>on

> \# service vsftpd restart

有关selinux的配置

如关闭，仅仅警告，强制等等 需要编辑/etc/sysconfig/selinux 默认是强制。

**1.553 Could not create file**
一般都是SELinux的问题，设置SELinux的一个值，重启服务器即可。

> setsebool -P ftpd\_disable\_trans 1
> service vsftpd restart

2.500 OOPS: bad bool value in config file for: write_enable
注意你的CentOS FTP服务文件中保证每一行最后没有任何空格，一般出错就是在多余的空格上。

这里也有一篇相关的文章：