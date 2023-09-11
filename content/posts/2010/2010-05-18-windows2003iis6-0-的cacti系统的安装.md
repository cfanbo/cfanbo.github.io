---
title: windows2003+iis6.0 的cacti系统的安装
author: admin
type: post
date: 2010-05-18T02:17:12+00:00
url: /archives/3656
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - CACTI

---
官方教程：

>

一、所需软件及下载链接：

1、Cacti
下载地址： [http://www.cacti.net/downloads/](http://www.cacti.net/downloads/)

（这个是Cacti的网页显示程序，是用PHP做的，完成之后你要把放他放在你的WEB目录里。）
2、Cactid
下载地址： [http://www.cacti.net/downloads/cactid/packages/Windows/](http://www.cacti.net/downloads/cactid/packages/Windows/)

（这个是cacti从RRDtool那里得到的图形生成图形的程序。）

Spine这个是Cactid的新版。0.8.6版之后就用这个做为生成图形与网页的接口了
3、RRDTool
下载地址： [http://www.cacti.net/downloads/rrdtool/win32/](http://www.cacti.net/downloads/rrdtool/win32/)

（这个就是生成图形的程序了，这个要用到cmd.exe程序。当然你要把你的cmd.exe加上USER权限）
4、PHP 4.3.6或5.x
下载地址： [http://www.php.net/downloads.php](http://www.php.net/downloads.php)
5、MySQL 4.x或MySQL 5.x
下载地址： [http://dev.mysql.com/downloads/](http://dev.mysql.com/downloads/)
6、(非必要) Cygwin
下载地址： [http://cygwin.com/](http://cygwin.com/)
7、Net-SNMP
下载地址： [http://net-snmp.sourceforge.net/](http://net-snmp.sourceforge.net/)
8、(非必要)ActivePerl – 如果您要执行perl档的话，请安装它.
下载地址： [http://www.activestate.com/Products/Download/Download.plex?id=ActivePerl](http://www.activestate.com/Products/Download/Download.plex?id=ActivePerl)

二、IIS+PHP5的安装配置

安装程序安装
PHP 的安装程序可以在 http://www.php.net/downloads.php 下载。点击 PHP zip 即可下载。

虽然目前有很多多合一的安装包，而且也发布了一个 Microsoft Windows 的 PHP 安装程序，但是仍然建议用户花些时间自己手动安装 PHP。
因为这样才可以更加了解这套系统，并能够在需要的时候更方便的安装 PHP 扩展。
同时服务器模块比 CGI 可执行程序提供了更好的性能和更多的功能。
CLI 版本是为使用 PHP 命令行脚本而设计的。
CGI 和 CLI 可执行文件以及 web 服务器模块都需要 php5ts.dll 。
必须确认该文件可以在 PHP 安装路径中找到。对该 DLL 的搜索顺序为：

1、调用 php.exe 时所在的目录，或者若使用 ISAPI 模块时，web 服务器的目录（例如 C:\Program Files\Apache Group\Apache2\bin）。

2、任何在 Windows 的 PATH 环境变量中指定的目录。

要让 php5ts.dll 能正确被搜索到，有下面三个选择：复制该文件到 Windows 系统目录，复制该文件到 web 服务器的目录，或者把 PHP 目录（例如 d:\PHP）添加到 PATH 环境变量中。为了将来更好的维护，建议使用最后一个选择，将 PHP 目录添加到 PATH 环境变量中，因为这样更便于将来升级 PHP。
下面介绍PHP手工安装步骤：

第一步：我是直接解压缩放到d盘PHP目录下了，这样查找文件会方便许多。解压缩完后我的PHP目录就是d:\PHP。NTFS文件系统 (XP/2000/2003特有)请为USERS设置文件夹的权限为系统默认的即可，否则运行PHP文件之后会出现服务器内部错误的提示。

第二步：将 PHP 目录添加到 PATH 环境变量中，

（我的电脑->属性->高级->环境变量->系统变量->找到 path 这个变量，点击编辑在后面加入 如 d:\php; 就是你安装PHP的路径 注意每一个变量之间有一个“;”半角的分号分隔，如果前面的没有分号请大家加上去。->一路确定）

第三步：为 PHP 设置一个有效的配置文件，php.ini。

在 ZIP 包中有两个 ini 文件，php.ini-dist 和 php.ini-recommended。

建议使用 php.ini-recommended，因为在该文件中优化了性能和安全。

请仔细阅读该文件中的注释，因为它从 php.ini-dist 修改而来，会对设置产生较大的影响。

例如将 display\_errors 设置为 off，将 magic\_quotes_gpc 设置为 off。

除了阅读这些部分，还可以学习一下 ini 设置，并手动设置每一个配置项目。

如果想要最安全的设置，这是最好的方法，虽然 PHP 在默认配置下也是很安全的。

先将D:\PHP\php.ini-recommended 重命名为 php.ini 。

查找register_globals = off 有一处(旧版PHP有2处)(根据自己需要修改 是否注册全局变量)

查找short\_open\_tag = Off，把off改成On 有一处，

查找extension\_dir = “./” 改为 extension\_dir = “d:\PHP\ext”

（指定动态连接库的目录，php5和php4不同的地方就是它的动态连接库目录变了，这在它的文档结构里有详细的说明）

然后再查找;extension=php_mbstring.dll，把下面几句前面的分号去掉

extension=php_mbstring.dll　这个不选的话用phpMyAdmin会出现红色提示
extension=php_dba.dll
extension=php_dbase.dll
extension=php_filepro.dll　可选
extension=php_gd2.dll　支持GD库的
extension=php_imap.dll 可选
extension=php_ldap.dll
extension=php_mysql.dll　支持MySQL的

extension=php_snmp.dll
extension=php_sockets.dll
cgi.force_redirect = 0

其他扩展请根据自己需要修改

接下来修改了一些文件上传以及内存使用最大限制：
memory_limit = 128M ;最大内存使用
post\_max\_size = 20M
upload\_max\_filesize = 2M ;附件大小

以上3个地方请大家根据自己的实际需要修改

别的就没改什么了，保存后退出。

第四步：使 php.ini文件在 Windows 下被 PHP 所用，

（我的电脑->属性->高级->环境变量->系统变量->点击“添加”->变量名“PHPRC”->变 量值“D:\PHP;”也就是你安装PHP的路径->一路确定）

重启计算机后变量设置生效，再进行以下步骤的操作。

打开internet信息服务管理器,进入IIS站点的管理 添加一个新的ISAPI筛选

名称我们可以随便输入一个,为了以后方便记忆,我输入的是 php, 在下面选择php5isapi.dll这个文件的位置.

再打开主目录标签.点击配置,在缓存ISAPI应用程序里面新建一个 扩展名為.PHP的 可执行文件的位置我们还是选择 php5isapi.dll 这个文件的位置.确定即可!

註意:如果你是WINDOWS 2003的系统,还需要在 web应用程序扩展里面添加一个新的扩展,
一样的选择php5isapi.dll这个文件的位置, 以前有的朋友会选择”所有未知ISAPI扩展为允许”其实我们现在这样做更为安全.

重新启动IIS服务.

现在我们的IIS已经能对.php的文件进行解释了.

二、安装 RRDTool
下载 RRDTool zip 档案从下面网站
[http://www.cacti.net/downloads/rrdtool/win32/](http://www.cacti.net/downloads/rrdtool/win32/)
并将它解压缩，复制数据夹里的数据到 c:\cacti .
三、安装 Net-Snmp
从网站 [http://net-snmp.sourceforge.net/](http://net-snmp.sourceforge.net/) 下载最新版本的Win32档案
并将它安装在 c:\net-snmp 下面
四、启动本机 SNMP
如果您也要侦测本机的snmp状态请启用它
开启控制台 → 新增移除程序 → 新增移除Windows组件 → Management and Monitoring Tools → Simple Network Management Protocol
将它打勾后点选确定并启动它.
五、安装 Cactid
下载 最新版本的 Cactid 从下面网站
[http://www.cacti.net/downloads/cactid/packages/Windows/](http://www.cacti.net/downloads/cactid/packages/Windows/)
解压缩 Cactid zip 档案，复制数据夹里的数据到 c:\cacti，并确定 cactid.conf 档案里的下面数据符合您的MySQL信息.
DB_Host         127.0.0.1 or hostname (请勿输入 localhost)
DB_Database      cacti
DB_User          cactiuser
DB_Password      cacti
DB_Port          3306
六、安装ActivePerl
请到下面网站下载最新版本的ActivePerl for Windows
[http://www.activestate.com/Products/Download/Download.plex?id=ActivePerl](http://www.activestate.com/Products/Download/Download.plex?id=ActivePerl)
请下载5.6.x.xxx版本
七、设定 Cacti
下载最新版本的 Cacti 从下面网站
[http://www.cacti.net/downloads/](http://www.cacti.net/downloads/)
解压缩档案后将档案复制到您的网页目录
MySQL 里新增一个 cacti 的数据库 然后汇入 cacti\_web\_root/cacti/cacti.sql 这一个档案
修改 cacti\_web\_root/cacti/include/config.php 这一个档案，并符合您的 MySQL 信息.
$database_default = “cacti”;
$database_hostname = “localhost”;
$database_username = “cactiuser”;
$database_password = “cacti”;
$database_port = “3306″;
八、打开您的浏览器输入下面网址:
[http://your-server/cacti/install](http://your-server/cacti/install)
并依照指示选择 New Install 然后点选下一步
之后这里需输入一写信息，如rrdtool、php、snmpwalk、snmpget的位置，请依照您上面安装路径输入正确的路径
所有路径都是此档案的绝对路径而不是所在目录
如果事后无法显示出图形请到Configuration → Settings → General
→ RRDTool Utility Version 将它改成RRDTool 1.2x
如果有图确没文字的话，请到paths里的RRDTool Default Font Path – c:/windows/fonts/arial.ttf
官方推荐的路径
php5：c:\php\php-win.exe
RRDTool Binary Path:
c:\rrdtool\rrdtool.exe.
SNMPGET, SNMPWALK Paths:
c:\net-snmp\bin\snmpwalk.exe
c:\net-snmp\bin\snmpget.exe
Cacti Logfile Path:
c:\website\cacti\log\cacti.log
Cactid Poller File Path:
c:\cactid\
九、登入的账号密码
登入的账号密码预设都是 admin. 登入后需立即更改您的密码。
十、定时执行命令
请打开您的命令提示符
输入下面
c:/php/php.exe c:/cacti\_web\_root/cacti/poller.php
测试是否有输出下面类似信息，如果是PHP5使用php_win.exe
C:\>c:/php/php.exe c:/cacti\_web\_root/cacti/poller.php
OK u:0.00 s:0.06 r:1.32
OK u:0.00 s:0.06 r:1.32
OK u:0.00 s:0.16 r:2.59
OK u:0.00 s:0.17 r:2.62
10/28/2005 04:57:12 PM – SYSTEM STATS: Time:4.7272 Method:cmd.php Processes:1 Threads:N/A Hosts:1 HostsPerProcess:2 DataSources:4 RRDsProcessed:2
之后您应该确认 cacti.log 档案有在 /cacti/log/出现跟 *.rrd 档案有在   /cacti/rra/ 出现.
点选开始 → 设定 → 控制台 → 排定的工作
点新增排定工作 → 下一步 → 点选浏览 → 并选择 C:\PHP\php.exe
输入排程名称 选择每日执行 →   下一步
这边不要变更 → 下一步
输入执行者的名称及密码 → 下一步
完成 → 勾起按下[完成]后开启这项工作的进阶内容
选择上方选项里的 → 工作 将执行里的指令改成(请注意您的poller.php档案的位置)
c:/php/php.exe c:/cacti\_web\_root/cacti/poller.php
选择上方选项里的 → 排程 点选进阶
勾选 重复执行 → 每隔改成5分钟 → 直到：改成期间 24小时 0 分钟
十、开始设定
现在您可以立即联机到cacti去设定了
如果配置好后不显示图，请在CACTI的磁盘根目录加USERS组的读取权限。

十一,一定不要忘记给cmd.exe加上user权限.要不图像显示不出来..这点还是亿恩老大提醒…

十二.还有一个..差点儿忘记.

cacti最新版本是0.8.7a，我原来用的是0.8.6j。因为自己修改了很多页面，进行批量添加主机之类的操作，所以一直没有去升级。
现在既然cacti已经有cli命令，那么就放弃我原来的东西，直接用官方的好了。
碰到了两个乱码的问题：
1、升级以后，发现中文全部显示不了，包括图像的中文。搞了半天，发现原来是数据库用的是GB2312，之前的cacti版本我修改过页面，不过现在忘记 了。那么，在lib/database.php文件中的“db\_connect\_real”函数里面加入”set names gb2312″即可。
修改完以后的“db\_connect\_real”函数：
`

function db_connect_real($host,$user,$pass,$db_name,$db_type, $port = "3306", $retries = 20) {         global $cnn_id;         $i = 0;         $cnn_id = NewADOConnection($db_type);         $hostport = $host . ":" . $port;         while ($i <= $retries) {                 if ($cnn_id->PConnect($hostport,$user,$pass,$db_name)) {                         $sql = "set names gb2312";                         $query = $cnn_id->Execute($sql);                         return(1);                 }                 $i++;                 usleep(40000);         }         die("FATAL: Cannot connect to MySQL server on '$host'. Please make sure you have specified a valid MySQL database name in 'include/config.php'\n");         return(0); }

`
2、完了以后发现”data source”的页面的中文还是乱码，直接编辑页面”data_source.php”，删除了”htmlentities”函数之后（大概在1150行 左右）恢复正常，如下：
修改前：
`

form_selectable_cell(" [" . (($_REQUEST["filter"] != "") ? eregi_replace("(" . preg_quote($_REQUEST["filter"]) . ")", "\\1", title_trim(htmlentities($data_source["name_cache"]), read_config_option("max_title_data_source"))) : title_trim(htmlentities($data_source["name_cache"]), read_config_option("max_title_data_source"))) . "](data_sources.php?action=ds_edit&id=" .  $data_source["local_data_id"] . ")", $data_source["local_data_id"]);

`
修改后
`

form_selectable_cell(" [" . (($_REQUEST["filter"] != "") ? eregi_replace("(" . preg_quote($_REQUEST["filter"]) . "     )", "\\1", title_trim($data_source["name_cache"], read_config_option("ma     x_title_data_source"))) : title_trim($data_source["name_cache"], read_config_option("max_title_data_source"))) . "](data_sources.php?action=ds_edit&id=" . $data      _source["local_data_id"] . ")",      $data_source["local_data_id"]);

`