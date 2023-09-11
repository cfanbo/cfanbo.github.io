---
title: centos5下vsftpd的设置（虚拟用户）–转
author: admin
type: post
date: 2010-12-27T10:45:13+00:00
url: /archives/7295
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos

---
### 本地用户经过设置后可以进行**ftp**访问。而匿名用户的访问经过了转换，在系统中。匿名用户的用户名为ftp, 系统将其属性设置为 根目录 /var/ftp/, 禁止控制台登陆，也就是，该用户只能进行ftp访问。

**CentOS**下**vsftpd** 的执行程序为 /etc/vsftpd，修改 /etc/vsftpd/vsftpd.conf文件中的listen要设置为YES.

VSFTPD有两种开机自启动模式: inet模式和standalone模式，推荐使用standalone模式。

在CentOS中已集成了VSFTPD软件。VSFTPD是一个安全高效的FTP服务软件，得到了广泛的应用。

**一、vsftpd 安装**

在服务中查看是否已安装VSFTPD服务。如没有，下载并安装：

rpm -ivh vsftpd-2.0.5-12.el5.i386.rpm

**二、设置vsftpd自启动**

chkconfig –level 35 vsftpd on

**三、vsftpd配置**

1 打开 /etc/vsftpd/vsftpd.conf文件。将anonymous\_enable=YES，改为anonymous\_enable=NO

2 打开 /etc/vsftpd/vsftpd.conf文件。添加user\_config\_dir=/etc/vsftpd/virtual，并建立virtual目录。在此目录中建立以用户名为文件名的文件，并写入：local_root=[目录]，这个目录即是FTP连接时的主目录。

3 限定用户只在自己目录：修改vsftpd.conf文件，取消注释：

chroot\_list\_enable=YES

chroot\_list\_file=/etc/vsftpd/chroot_list

在/etc/vsftpd/目录下添加文件chroot_list，加入作为FTP用户的本地用户名。

4 解决用户无法进入目录问题：

打开终端，输入：setsebool -P ftpd\_disable\_trans 1

然后重启FTP服务：service vsftpd restart

**四、权限：**

假设是/var/www/html

这个目录的权限应该是770，owner是root，group是ftp

chmod 770 /var/www/html

chown root:ftp /var/www/html

### centos5下vsftpd的设置（虚拟用户）

前面学校好几台服务器上都需要配置ftp，但一直没有整成功，这次总算完成一件事情了，在centos下完成vsftp的配置了。

步骤很简单：

需求:（虚拟用户分下载用户／下载、上传但不能删除用户／管理用户）

**一、安装**
yum -y install vsftpd*
yum -y install pam*
yum -y install db4*

**二、系统帐户**

1、vsftpd服务的宿主用户
useradd vsftpd -s /sbin/nologin
默认的Vsftpd的服务宿主用户是root，但是这不符合安全性的需要。这里建立名字为vsftpd的用户，用他来作为支持Vsftpd的服务宿主用户。由于该用户仅用来支持Vsftpd服务用，因此没有许可他登陆系统的必要，并设定他为不能登陆系统的用户。

2、vsftpd虚拟宿主用户
useradd ftp -s /sbin/nologin(服务器上装完了就用一个用户是ftp)

本篇主要是介绍Vsftp的虚拟用户，虚拟用户并不是系统用户，也就是说这些FTP的用户在系统中是不存在的。他们的总体权限其实是集中寄托在一个在系统 中的某一个用户身上的，所谓Vsftpd的虚拟宿主用户，就是这样一个支持着所有虚拟用户的宿主用户。由于他支撑了FTP的所有虚拟的用户，那么他本身的 权限将会影响着这些虚拟的用户，因此，处于安全性的考虑，也要非分注意对该用户的权限的控制，该用户也绝对没有登陆系统的必要，这里也设定他为不能登陆系 统的用户。

*不允许相关用户登录。
**三、vsftpd.conf设置**
1、备份
cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak
2、设置
—-
anonymous_enable=NO
设定不允许匿名访问
local_enable=YES
设定本地用户可以访问。注意：主要是为虚拟宿主用户，如果该项目设定为NO那么所有虚拟用户将无法访问。
write_enable=YES
设定可以进行写操作。
local_umask=022
设定上传后文件的权限掩码。
anon\_upload\_enable=NO
禁止匿名用户上传。
anon\_mkdir\_write_enable=NO
禁止匿名用户建立目录。
dirmessage_enable=YES
设定开启目录标语功能。
xferlog_enable=YES
设定开启日志记录功能。
connect\_from\_port_20=YES
设定端口20进行数据连接。
chown_uploads=NO
设定禁止上传文件更改宿主。
xferlog_file=/var/log/vsftpd.log
设定Vsftpd的服务日志保存路径。注意，该文件默认不存在。必须要手动touch出来，并且由于这里更改了Vsftpd的服务宿主用户为手动建立的Vsftpd。必须注意给与该用户对日志的写入权限，否则服务将启动失败。
xferlog\_std\_format=YES
设定日志使用标准的记录格式。
nopriv_user=vsftpd
设定支撑Vsftpd服务的宿主用户为手动建立的Vsftpd用户。注意，一旦做出更改宿主用户后，必须注意一起与该服务相关的读写文件的读写赋权问题。比如日志文件就必须给与该用户写入权限等。
async\_abor\_enable=YES
设定支持异步传输功能。
ascii\_upload\_enable=YES
ascii\_download\_enable=YES
设定支持ASCII模式的上传和下载功能。
ftpd_banner=Welcome to Awei FTP servers
设定Vsftpd的登陆标语。
chroot\_local\_user=YES
禁止本地用户登出自己的FTP主目录。
pam\_service\_name=vsftpd
设定PAM服务下Vsftpd的验证配置文件名。因此，PAM验证将参考/etc/pam.d/下的vsftpd文件配置。
以下这些是关于Vsftpd虚拟用户支持的重要配置项目。默认Vsftpd.conf中不包含这些设定项目，需要自己手动添加配置。
guest_enable=YES
设定启用虚拟用户功能。
guest_username=ftp
指定虚拟用户的宿主用户。
virtual\_use\_local_privs=YES
设定虚拟用户的权限符合他们的宿主用户。
user\_config\_dir=/etc/vsftpd/vconf
设定虚拟用户个人Vsftp的配置文件存放路径。也就是说，这个被指定的目录里，将存放每个Vsftp虚拟用户个性的配置文件，一个需要注意的
地方就是这些配置文件名必须和虚拟用户名相同。\[color=Red\]\[b\]（比如说vsftpd.conf的配置文件，你复制到这个目录下，你要mv一下，配置成虚拟用户的名称）\[/b\]\[/color\]—-
3.建立Vsftpd的日志文件，并更该属主为Vsftpd的服务宿主用户：
[root@KcentOS5 ~]# touch /var/log/vsftpd.log
[root@KcentOS5 ~]# chown vsftpd.vsftpd /var/log/vsftpd.log
4.建立虚拟用户配置文件存放路径：
[root@KcentOS5 ~]# mkdir /etc/vsftpd/vconf/
**四、制作虚拟用户数据库文件**
1.先建立虚拟用户名单文件：
[root@KcentOS5 ~]# touch /etc/vsftpd/virtusers
建立了一个虚拟用户名单文件，这个文件就是来记录vsftpd虚拟用户的用户名和口令的数据文件，我这里给它命名为virtusers。为了避免文件的混乱，我把这个名单文件就放置在/etc/vsftpd/下。
2.编辑虚拟用户名单文件：
[root@KcentOS5 ~]# vi /etc/vsftpd/virtusers
—————————-
download
1234
upload
5678
admin
9012
—————————-
编辑这个虚拟用户名单文件，在其中加入用户的用户名和口令信息。格式很简单：“一行用户名，一行口令”。
3.生成虚拟用户数据文件：
[root@KcentOS5 ~]# db_load -T -t hash -f /etc/vsftpd/virtusers /etc/vsftpd/virtusers.db
**五、设定PAM验证文件，并指定虚拟用户数据库文件进行读取**
在/etc/pam.d/vsftpd的文件头部加入以下信息（在后面加入无效）
—-
auth sufficient /lib/security/pam_userdb.so db=/etc/vsftpd/virtusers
account sufficient /lib/security/pam_userdb.so db=/etc/vsftpd/virtusers
—-
**六、虚拟用户的配置**
local_root=/var/www/html
指定虚拟用户的具体主路径。
anonymous_enable=NO
设定不允许匿名用户访问。
write_enable=YES
设定允许写操作。
local_umask=022
设定上传文件权限掩码。
anon\_upload\_enable=NO
设定不允许匿名用户上传。
anon\_mkdir\_write_enable=NO
设定不允许匿名用户建立目录。
idle\_session\_timeout=600
设定空闲连接超时时间。
data\_connection\_timeout=120
设定单次连续传输最大时间。
max_clients=10
设定并发客户端访问个数。
max\_per\_ip=5
设定单个客户端的最大线程数，这个配置主要来照顾Flashget、迅雷等多线程下载软件。
local\_max\_rate=50000
设定该用户的最大传输速率，单位b/s。
——————————–
这里将原vsftpd.conf配置文件经过简化后保存作为虚拟用户配置文件的模版。这里将并不需要指定太多的配置内容，主要的框架和限制交由 Vsftpd的主配置文件vsftpd.conf来定义，即虚拟用户配置文件当中没有提到的配置项目将参考主配置文件中的设定。而在这里作为虚拟用户的配 置文件模版只需要留一些和用户流量控制，访问方式控制的配置项目就可以了。这里的关键项是local_root这个配置，用来指定这个虚拟用户的FTP主 路径。

这里有一个最主要的问题，就是目录的宿主和宿主用户不是虚拟用户，我们设置了目录后还只能下载，不能上传和下载，如果想上传就要使用chown的命令
chmod o+w /var/www/html/ o是指其它的用户，w是写的权限

以上是个人的配置心得，大部门都是互联网上的，但查询很多资料，是找不到个人的心得的，哈哈

# [centos 安装VSFTP][1]{#AjaxHolder_ctl01_TitleUrl}

1.此次为了测试了解一下,VSFTP,采用RPM包安装方式


首先


rpm -qa | grep vsftpd           ———查看有无安装,若没有,则要安装,我采用的是yum


yum install vsftpd


见下:


[root@ftp sbin]# yum install vsftpd


[root@ftp sbin]# service vsftpd status

vsftpd is stopped

[root@ftp sbin]# service vsftpd start

Starting vsftpd for vsftpd:                                [ OK ]

[root@ftp sbin]#


2.设置每次开机时自动运行及手工启动它:


chkconfig vsftpd on


service vsftpd start


netstat -tl    可以查看ftp端口是否在侦听了!


相关配置文件:/etc/vsftpd/vsftpd.conf里面;


3.至此已经可以FTP已经可以正常运行了,


4.FTP配置参考以下设置:


初级测试篇:(注:匿名用户使用ftp这个系统用户,无需密码)


a． 匿名服务器的连接（独立的服务器）

在/etc/vsftpd/vsftpd.conf配置文件中添加如下几项：

Anonymous_enable=yes (允许匿名登陆)

Dirmessage_enable=yes （切换目录时，显示目录下.message的内容）

Local_umask=022 (FTP上本地的文件权限，默认是077)

Connect_form_port_20=yes （启用FTP数据端口的数据连接）*

Xferlog_enable=yes （激活上传和下载的日志）

Xferlog_std_format=yes (使用标准的日志格式)

Ftpd_banner=XXXXX （欢迎信息）

Pam_service_name=vsftpd （验证方式）*

Listen=yes （独立的VSFTPD服务器）*

功能：只能连接FTP服务器，不能上传和下载


注：其中所有和日志欢迎信息相关连的都是可选项,打了星号的无论什么帐户都要添加，是属于FTP的基本选项

b． 开启匿名FTP服务器上传权限

在配置文件中添加以下的信息即可：

Anon_upload_enable=yes (开放上传权限)

Anon_mkdir_write_enable=yes （可创建目录的同时可以在此目录中上传文件）

Write_enable=yes (开放本地用户写的权限)

Anon_other_write_enable=yes (匿名帐号可以有删除的权限)


c． 开启匿名服务器下载的权限

在配置文件中添加如下信息即可：

Anon_world_readable_only=no

注：要注意文件夹的属性，匿名帐户是其它（other）用户要开启它的读写执行的权限

（R）读—–下载 （W）写—-上传 （X）执行—-如果不开FTP的目录都进不去


d．普通用户FTP服务器的连接（独立服务器）

在配置文件中添加如下信息即可：

Local_enble=yes （本地帐户能够登陆）

Write_enable=no （本地帐户登陆后无权删除和修改文件）

功能：可以用本地帐户登陆vsftpd服务器，有下载上传的权限

注：在禁止匿名登陆的信息后匿名服务器照样可以登陆但不可以上传下载


e． 用户登陆限制进其它的目录，只能进它的主目录

设置所有的本地用户都执行chroot

Chroot_local_user=yes （本地所有帐户都只能在自家目录）

设置指定用户执行chroot

Chroot_list_enable=yes （文件中的名单可以调用）

Chroot_list_file=/任意指定的路径/vsftpd.chroot_list

注意：vsftpd.chroot_list 是没有创建的需要自己添加，要想控制帐号就直接在文件中加帐号即可

f． 限制本地用户访问FTP

Userlist_enable=yes (用userlistlai 来限制用户访问)

Userlist_deny=no (名单中的人不允许访问)

Userlist_file=/指定文件存放的路径/ （文件放置的路径）

注：开启userlist_enable=yes匿名帐号不能登陆


g． 安全选项

Idle_session_timeout=600(秒) （用户会话空闲后10分钟）

Data_connection_timeout=120（秒） （将数据连接空闲2分钟断）

Accept_timeout=60（秒） （将客户端空闲1分钟后断）

Connect_timeout=60（秒） （中断1分钟后又重新连接）

Local_max_rate=50000（bite） （本地用户传输率50K）

Anon_max_rate=30000（bite） （匿名用户传输率30K）

Pasv_min_port=50000 （将客户端的数据连接端口改在

Pasv_max_port=60000 50000—60000之间）

Max_clients=200 （FTP的最大连接数）

Max_per_ip=4 （每IP的最大连接数）

Listen_port=5555 （从5555端口进行数据连接）


h． 查看谁登陆了FTP,并杀死它的进程

ps –xf |grep ftp

kill 进程号


5. 高阶部分测试篇:


配置本地组访问的FTP

首先创建用户组 test和FTP的主目录


groupadd test


mkdir /tmp/test


然后创建用户


useradd -G test –d /tmp/test –M usr1


注：G：用户所在的组 d：表示创建用户的自己目录的位置给予指定


M：不建立默认的自家目录，也就是说在/home下没有自己的目录


useradd –G test –d /tmp/test –M usr2


接着改变文件夹的属主和权限


chown usr1.test /tmp/test —-这表示把/tmp/test的属主定为usr1


chmod 750 /tmp/test —-7表示wrx 5表示rx 0表示什么权限都没有


这个实验的目的就是usr1有上传、删除和下载的权限


而usr2只有下载的权限没有上传和删除的权限


当然啦大家别忘了我们的主配置文件vsftpd.conf


———————————————————————–


修改用户密码或添加用户密码


以用户name为例，添加用户：useradd name,设置密码：passwd name,然后根据提示，输入两次密码即可。


删除用户：userdel name,其实并没有完全删除，只是该用户不能登陆，其目录下的文件还在保留。


如:useradd username


passwd username


userdel username


—————————————————————


要确定local_enable=yes、write_enable=yes、chroot_local_usr=yes这三个选项是有的!


6. VSFTPD.conf里面的参数简要说明:

Anonymous_enable=yes (允许匿名登陆)


Dirmessage_enable=yes （切换目录时，显示目录下.message的内容）


Local_umask=022 (FTP上本地的文件权限，默认是077)


Connect_form_port_20=yes （启用FTP数据端口的数据连接）*


Xferlog_enable=yes （激活上传和下传的日志）


Xferlog_std_format=yes (使用标准的日志格式)


Ftpd_banner=XXXXX （欢迎信息）


Pam_service_name=vsftpd （验证方式）*


Listen=yes （独立的VSFTPD服务器）*


Anon_upload_enable=yes (开放上传权限)


Anon_mkdir_write_enable=yes （可创建目录的同时可以在此目录中上传文件）


Write_enable=yes (开放本地用户写的权限)


Anon_other_write_enable=yes (匿名帐号可以有删除的权限)


Anon_world_readable_only=no (放开匿名用户浏览权限)


Ascii_upload_enable=yes (启用上传的ASCII传输方式)


Ascii_download_enable=yes (启用下载的ASCII传输方式)


Banner_file=/var/vsftpd_banner_file (用户连接后欢迎信息使用的是此文件中的相关信息)


Idle_session_timeout=600(秒) （用户会话空闲后10分钟）


Data_connection_timeout=120（秒） （将数据连接空闲2分钟断）


Accept_timeout=60（秒） （将客户端空闲1分钟后断）


Connect_timeout=60（秒） （中断1分钟后又重新连接）


Local_max_rate=50000（bite） （本地用户传输率50K）


Anon_max_rate=30000（bite） （匿名用户传输率30K）


Pasv_min_port=50000 （将客户端的数据连接端口改在


Pasv_max_port=60000 50000—60000之间）


Max_clients=200 （FTP的最大连接数）


Max_per_ip=4 （每IP的最大连接数）


Listen_port=5555 （从5555端口进行数据连接）


Local_enble=yes （本地帐户能够登陆）


Write_enable=no （本地帐户登陆后无权删除和修改文件）


这是一组


Chroot_local_user=yes （本地所有帐户都只能在自家目录）


Chroot_list_enable=yes （文件中的名单可以调用）


Chroot_list_file=/任意指定的路径/vsftpd.chroot_list


（前提是chroot_local_user=no）


这又是一组


Userlist_enable=yes （在指定的文件中的用户不可以访问）


Userlist_deny=yes


Userlist_file=/指定的路径/vsftpd.user_list


又开始单的了


Banner_fail=/路径/文件名 （连接失败时显示文件中的内容）


Ls_recurse_enable=no


Async_abor_enable=yes


One_process_model=yes


Listen_address=10.2.2.2 （将虚拟服务绑定到某端口）


Guest_enable=yes (虚拟用户可以登陆)


Guest_username=所设的用户名 （将虚拟用户映射为本地用户）


User_config_dir=/任意指定的路径/为用户策略自己所建的文件夹


(指定不同虚拟用户配置文件的路径)


又是一组


Chown_uploads=yes （改变上传文件的所有者为root）


Chown_username=root


又是一组


Deny_email_enable=yes (是否允许禁止匿名用户使用某些邮件地址)


Banned_email_file=//任意指定的路径/xx/


又是单的


Pasv_enable=yes （ 服务器端用被动模式）


User_config_dir=/任意指定的路径//任意文件目录 (指定虚拟用户存放配置文件的路径)


 [1]: http://www.cnblogs.com/rooney/archive/2009/03/14/1411620.html