---
title: FREEBSD:VSFTP的安装和配置(packages方式)
author: admin
type: post
date: 2008-10-29T12:27:43+00:00
excerpt: |
 |
 一、预备工作：
 1.新建目录
 mkdir /usr/share/empty
 mkdir /var/ftp
 2.改变目录所有者和权限
 chown root:wheel /var/ftp（如果是Linux用chown root:root /var/ftp）
 chmod og-w /var/ftp （此命令是取消其他用户的写权限）
 二、安装VSFTP
 1.用tar包安装
 tar zvf vsftpd-2.0.1.gz.tar
 cd vsfpd-2.0.1
 make
 make install
 2．用ports安装（只适合FREEBSD，而且必须是可以上网的用户，对Linux用户不适用）
 cd /usr/ports/ftp/vsftpd
 make
 make install
url: /archives/488
IM_data:
 - 'a:1:{s:38:"/wp-content/uploads/2009/01/vsftp1.jpg";s:38:"/wp-content/uploads/2009/01/vsftp1.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - vsftpd

---
一、预备工作：
1.新建目录
mkdir /usr/share/empty
mkdir /var/ftp
2.改变目录所有者和权限
chown root:wheel /var/ftp（如果是Linux用chown root:root /var/ftp）
chmod og-w /var/ftp （此命令是取消其他用户的写权限）
二、安装VSFTP
1.用tar包安装
tar zvf vsftpd-2.0.1.gz.tar
cd vsfpd-2.0.1
make
make install
2．用ports安装（只适合FREEBSD，而且必须是可以上网的用户，对Linux用户不适用）
cd /usr/ports/ftp/vsftpd
make
make install
安装的时候会弹出一个对话框,

[![](https://blogstatic.haohtml.com//uploads/2023/09/vsftp1.jpg)][1]
选中第一个选项项
三、配置
1．配置VSFTP
打开/etc/vsftpd.conf，(如果用ports安装的话是在/usr/local/etc/vsftpd.conf)，,相关参数说明如下:
＝＝＝个别使用者设定＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
chroot\_list\_enable
用法：YES/NO
如果启动这项功能，则所有的本机使用者登入均可进到根目录之外的数据夹，除了列在/etc/vsftpd.chroot_list 之中的使用者之外。默认值为NO。
userlist_enable
用法：YES/NO
若是启动此功能，则会读取/etc/vsftpd.user_list 当中的使用者名称。此项功能可以在询问密码前就出现失败讯息，而不需要检验密码的程序。默认值为关闭。
userlist_deny
用法：YES/NO
这个选项只有在userlist\_enable 启动时才会被检验。如果将这个选项设为YES，则在/etc/vsftpd.user\_list 中的使用者将无法登入 若设为NO ， 则只有在/etc/vsftpd.user_list 中的使用者才能登入。而且此项功能可以在询问密码前就出现错误讯息，而不需要检验密码的程序。
user\_config\_dir
定义个别使用者设定文件所在的目录，例如定义user\_config\_dir=/etc/vsftpd/userconf，且主机上有使用者test1,test2，那我们可以在user\_config\_dir 的目录新增文件名为test1 以及test2。若是test1 登入，则会读取user\_config\_dir 下的test1 这个档案内的设定。默认值为无。
＝＝＝欢迎语设定＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
dirmessage_enable
如果启动这个选项，使用者第一次进入一个目录时，会检查该目录下是否有.message这个档案，若是有，则会出现此档案的内容，通常这个档案会放置欢迎话语，或是对该目录的说明。默认值为开启。
banner_file
当使用者登入时，会显示此设定所在的档案内容，通常为欢迎话语或是说明。默认值为无。
ftpd_banner
这边可定义欢迎话语的字符串，相较于banner\_file 是档案的形式，而ftpd\_banner 是字串的格式。预设为无。
＝＝＝特殊安全设定＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
chroot\_local\_user
如果设定为YES，那么所有的本机的使用者都可以切换到根目录以外的数据夹。预设值为NO。
hide_ids
如果启动这项功能，所有档案的拥有者与群组都为ftp，也就是使用者登入使用ls -all之类的指令，所看到的档案拥有者跟群组均为ftp。默认值为关闭。
ls\_recurse\_enable
若是启动此功能，则允许登入者使用ls -R 这个指令。默认值为NO。
write_enable
用法：YES/NO
这个选项可以控制FTP 的指令是否允许更改file system，譬如STOR、DELE、RNFR、RNTO、MKD、RMD、APPE 以及SITE。预设是关闭。
setproctitle_enable
用法：YES/NO
启动这项功能，vsftpd 会将所有联机的状况已不同的process 呈现出来，换句话说，使用ps -ef 这类的指令就可以看到联机的状态。默认值为关闭。
tcp_wrappers
用法：YES/NO
如果启动，则会将vsftpd 与tcp wrapper 结合，也就是可以在/etc/hosts.allow 与/etc/hosts.deny 中定义可联机或是拒绝的来源地址。
pam\_service\_name
这边定义PAM 所使用的名称，预设为vsftpd。
secure\_chroot\_dir
这个选项必须指定一个空的数据夹且任何登入者都不能有写入的权限，当vsftpd 不需要file system 的权限时，就会将使用者限制在此数据夹中。默认值为/usr/share/empty
＝＝＝纪录文件设定＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
xferlog_enable
用法：YES/NO
如果启动，上传与下载的信息将被完整纪录在底下xferlog_file 所定义的档案中。预设为开启。
xferlog_file
这个选项可设定纪录文件所在的位置，默认值为/var/log/vsftpd.log。
xferlog\_std\_format
如果启动，则纪录文件将会写为xferlog 的标准格式，如同wu-ftpd 一般。默认值为关闭。
＝＝＝逾时设定＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
accept_timeout
接受建立联机的逾时设定，单位为秒。默认值为60。
connect_timeout
响应PORT 方式的数据联机的逾时设定，单位为秒。默认值为60。
data\_connection\_timeout
建立数据联机的逾时设定。默认值为300 秒。
idle\_session\_timeout
发呆的逾时设定，若是超出这时间没有数据的传送或是指令的输入，则会强迫断线，单位为秒。默认值为300。
＝＝＝速率限制＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
anon\_max\_rate
匿名登入所能使用的最大传输速度，单位为每秒多少bytes，0 表示不限速度。默认值为0。
local\_max\_rate
本机使用者所能使用的最大传输速度，单位为每秒多少bytes，0 表示不限速度。预设值为0。
＝＝＝新增档案权限设定＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
anon_umask
匿名登入者新增档案时的umask 数值。默认值为077。
file\_open\_mode
上传档案的权限，与chmod 所使用的数值相同。默认值为0666。
local_umask
本机登入者新增档案时的umask 数值。默认值为077。
＝＝＝port 设定＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
connect\_from\_port_20
用法：YES/NO
若设为YES，则强迫ftp-data 的数据传送使用port 20。默认值为YES。
ftp\_data\_port
设定ftp 数据联机所使用的port。默认值为20。
listen_port
FTP server 所使用的port。默认值为21。
pasv\_max\_port
建立资料联机所可以使用port 范围的上界，0 表示任意。默认值为0。
pasv\_min\_port
建立资料联机所可以使用port 范围的下界，0 表示任意。默认值为0。
＝＝＝其它＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
anon_root
使用匿名登入时，所登入的目录。默认值为无。
local_enable
用法：YES/NO
启动此功能则允许本机使用者登入。默认值为YES。
local_root
本机使用者登入时，将被更换到定义的目录下。默认值为无。
text\_userdb\_names
用法：YES/NO
当使用者登入后使用ls -al 之类的指令查询该档案的管理权时，预设会出现拥有者的UID，而不是该档案拥有者的名称。若是希望出现拥有者的名称，则将此功能开启。默认值为NO。
pasv_enable
若是设为NO，则不允许使用PASV 的模式建立数据的联机。默认值为开启。
＝＝＝更换档案所有权＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
chown_uploads
用法：YES/NO
若是启动，所有匿名上传数据的拥有者将被更换为chown_username 当中所设定的使用者。这样的选项对于安全及管理，是很有用的。默认值为NO。
chown_username
这里可以定义当匿名登入者上传档案时，该档案的拥有者将被置换的使用者名称。预设值为root。
＝＝＝guest 设定＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
guest_enable
用法：YES/NO
若是启动这项功能，所有的非匿名登入者都视为guest。默认值为关闭。
guest_username
这里将定义guest 的使用者名称。默认值为ftp。
＝＝＝anonymous 设定＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
anonymous_enable
用法：YES/NO
管控使否允许匿名登入，YES 为允许匿名登入，NO 为不允许。默认值为YES。
no\_anon\_password
若是启动这项功能，则使用匿名登入时，不会询问密码。默认值为NO。
anon\_mkdir\_write_enable
用法：YES/NO
如果设为YES，匿名登入者会被允许新增目录，当然，匿名使用者必须要有对上层目录的写入权。默认值为NO。
anon\_other\_write_enable
用法：YES/NO
如果设为YES，匿名登入者会被允许更多于上传与建立目录之外的权限，譬如删除或是更名。默认值为NO。
anon\_upload\_enable
用法：YES/NO
如果设为YES，匿名登入者会被允许上传目录的权限，当然，匿名使用者必须要有对上层目录的写入权。默认值为NO。
anon\_world\_readable_only
用法：YES/NO
如果设为YES，匿名登入者会被允许下载可阅读的档案。默认值为YES。
ftp_username
定义匿名登入的使用者名称。默认值为ftp。
deny\_email\_enable
若是启动这项功能，则必须提供一个档案/etc/vsftpd.banner_emails，内容为email address。若是使用匿名登入，则会要求输入email address，若输入的email address 在此档案内，则不允许联机。默认值为NO。
＝＝＝Standalone 选项＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
listen
用法：YES/NO
若是启动，则vsftpd 将会以独立运作的方式执行，若是vsftpd 独立执行，如RedHat9的默认值，则必须启动 若是vsftpd 包含在xinetd 之中，则必须关闭此功能，如RedHat8。在RedHat9 的默认值为YES。
listen_address
若是vsftpd 使用standalone 的模式，可使用这个参数定义使用哪个IP address 提供这项服务，若是主机上只有定义一个IP address，则此选项不需使用，若是有多个IPaddress，可定义在哪个IP address 上提供ftp 服务。若是不设定，则所有的IP address均会提供此服务。默认值为无。
max_clients
若是vsftpd 使用standalone 的模式，可使用这个参数定义最大的总联机数。超过这个数目将会拒绝联机，0 表示不限。默认值为0。
max\_per\_ip
若是vsftpd 使用standalone 的模式，可使用这个参数定义每个ip address 所可以联机的数目。超过这个数目将会拒绝联机，0 表示不限。默认值为0。

2.添加用户（FREEBSD系统使用adduser命令，Linux系统用useradd）
1.新建匿名用户
adduser -d /var/ftp ftp
2.新建普通用户,这里就用test1
adduser -d /var/ftp test1 (Linux系统使用useradd -d /var/ftp）
四．启动服务
1．配置inetd
打开/etc/inetd.conf，并添加下面一行
ftp stream tcp nowait root /usr/sbin/tcpd /usr/local/sbin/vsftpd
现在重新启动inted服务，让配置生效
killall –HUP inetd

附：
FTP 数字代码的意义
110 重新启动标记应答。
120 服务在多久时间内ready。
125 数据链路埠开启，准备传送。
150 文件状态正常，开启数据连接端口。
200 命令执行成功。
202 命令执行失败。
211 系统状态或是系统求助响应。
212 目录的状态。
213 文件的状态。
214 求助的讯息。
215 名称系统类型。
220 新的联机服务ready。
221 服务的控制连接埠关闭，可以注销。
225 数据连结开启，但无传输动作。
226 关闭数据连接端口，请求的文件操作成功。
227 进入passive mode。
230 使用者登入。
250 请求的文件操作完成。
257 显示目前的路径名称。
331 用户名称正确，需要密码。
332 登入时需要账号信息。
350 请求的操作需要进一部的命令。
421 无法提供服务，关闭控制连结。
425 无法开启数据链路。
426 关闭联机，终止传输。
450 请求的操作未执行。
451 命令终止：有本地的错误。
452 未执行命令：磁盘空间不足。
500 格式错误，无法识别命令。
501 参数语法错误。
502 命令执行失败。
503 命令顺序错误。
504 命令所接的参数不正确。
530 未登入。
532 储存文件需要账户登入。
550 未执行请求的操作。
551 请求的命令终止，类型未知。
552 请求的文件终止，储存位溢出。
553 未执行请求的的命令，名称不正确

[1]: /wp-content/uploads/2009/01/vsftp1.jpg