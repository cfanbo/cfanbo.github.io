---
title: '[教程]freebsd下nagios安装教程'
author: admin
type: post
date: 2010-07-11T14:45:28+00:00
url: /archives/4579
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - nagios

---
一、安装apache

> cd /usr/ports/www/apache2
> make install clean

二、安装nagios和 Nagios-plugin

1.安装nagios主程序

> cd /usr/ports/net-mgmt/nagios
> make install clean

在安装的过程中会自动安装nagios-plugin插件.常用的脚本这时会被安装在” **/usr/local/libexec/nagios/**“路径在resource.cfg里面有定义的,即常量 $USER1$的值目录里.

2. NRpe  下载地址为： [http://www.nagios.org/download/addons](http://www.nagios.org/download/addons)

//如果所要监控的服务器中有linux，不要直接使用ports 安装，因为ports安装过后，其格式为check_nrpe2, 但是linux 采用源码安装的为check_nrpe. 从而导致两者之间不能通信

>

> #cd /usr/local

#fetch [http://prdownloads.sourceforge.net/sourceforge/nagios/nrpe-2.12.tar.gz](http://prdownloads.sourceforge.net/sourceforge/nagios/nrpe-2.12.tar.gz)
>

>
>

> #tar zxvf nrpe-2.12.tar.gz
>

>
>

> #cd nrpe-2.12
>

>
>

> #./configure
>

>
>

> #make
>

>
>

> #make install

#make install-plugin

#make install-daemon

#make install-daemon-config
>

>
>

> //启用服务

#/usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d

#/usr/local/nagios/libexec/check_nrpe -H localhost
>

四、修改http.conf

> DocumentRoot “/usr/local/www/nagios”
>
> ScriptAlias /nagios/cgi-bin /usr/local/www/nagios/cgi-bin
>
>
> Options ExecCGI
> AllowOverride None
> Order allow,deny
> Allow from all
>
> #nagios安全设置
> AuthType Basic
> AuthName “Nagios Access”
> AuthUserFile /usr/local/etc/nagios/htpasswd
> Require valid-user
>
>
> Alias /nagios/usr/local/www/nagios
>
> Options None
> AllowOverride None
> Order allow,deny
> Allow from all
> #nagios安全设置
> AuthType Basic
> AuthName “nagios Access”  //对话框右上角提示文字
> AuthUserFile /usr/local/etc/nagios/htpasswd //认证文件
> Require valid-user //认证用户名,上面用的是nagiosadmin
>

4、添加用户

> /usr/local/sbin/htpasswd -c /usr/local/etc/nagios/htpasswd test

上面是添加test用户，然后按提示输入密码

5、添加nagios配置文件
cp /usr/local/etc/nagios目录及其下objects目录里cfg-sample文件为cfg文件

nagios.cfg配置文件详解:

>

> cfg_file=/usr/local/nagios/etc/objects/commands.cfg             ＃ 命令的配置文件路径
>

>
>

> cfg_file=/usr/local/nagios/etc/objects/contacts.cfg             ＃ 联系人配置文件路径

cfg_file=/usr/local/nagios/etc/objects/contactgroups.cfg        ＃ 联系组配置文件路径

cfg_file=/usr/local/nagios/etc/objects/timeperiods.cfg          ＃ 监视时段配置文件路径

cfg_file=/usr/local/nagios/etc/objects/templates.cfg            ＃ 模板的配置文件路径

cfg_file=/usr/local/nagios/etc/objects/hostgroups.cfg           ＃ 主机组配置文件路径

cfg_file=/usr/local/nagios/etc/objects/hosts.cfg                ＃ 主机配置文件路径

cfg_file=/usr/local/nagios/etc/objects/services.cfg             ＃ 服务配置文件路径

＃把关于windows的配置选项前面的＃号去掉

cfg_file=/usr/local/nagios/etc/objects/windows.cfg

＃把server目录配置前面的＃去掉，记得手动创建目录

cfg_dir=/usr/local/nagios/etc/servers

＃如果需要查看日志就把下面的配置加上，记得自己手动创建目录

log_file=/usr/local/nagios/var/nagios.log

debug_file=/usr/local/nagios/var/nagios.debug

debug_level=32
>

6、修改 /usr/local/etc/nagios/cgi.cfg

> authorized\_for\_system_information=test
> authorized\_for\_configuration_information=test
> authorized\_for\_system_commands=test
> authorized\_for\_all_services=test
> authorized\_for\_all_hosts=nagiosadmin,test
> authorized\_for\_all\_service\_commands=test
> authorized\_for\_all\_host\_commands=test

按你需要添加用户权限，如果有多用户，请用逗号格开

7、执行下面命令测试nagios配置文件

> /usr/local/bin/nagios -v /usr/local/etc/nagios/nagios.cfg

8、启动nagios服务

> /usr/local/bin/nagios -d /usr/local/etc/nagios/nagios.cfg

9、启动apache2

> /usr/local/sbin/apachectl start

如果在使用中遇到以下错误:

It appears as though you do not have permission to view information for any of the hosts you requested…

If you believe this is an error, check the HTTP server authentication requirements for accessing this CGI
and check the authorization options in your CGI configuration file.

**解决办法:**

修改/usr/local/etc/nagios/cgi.cfg

> vi    /usr/local/etc/nagios/cgi.cfg

 use_authentication=1 #把1修改为0,保存

重启nagios即可.

最后在浏览器里输入nagios管理地址,会弹出以下对话框,输入用户名和密码即可.

[![](http://blog.haohtml.com/wp-content/uploads/2010/07/nagios_htpasswd.jpg)][1]

如果看到以下界面,说明安装成功了.

[![](http://blog.haohtml.com/wp-content/uploads/2010/07/nagios.png)][2]

**相关教程:**

监控Linux主机配置教程:,被监控端（被监控的服务器）安装nagios-nrpe_2.12.tar.gz和插件nagios-plugins-1.4.13.tar.gz

Centos下安装Nagiso监控系统教程:

参考:

 [1]: http://blog.haohtml.com/wp-content/uploads/2010/07/nagios_htpasswd.jpg
 [2]: http://blog.haohtml.com/wp-content/uploads/2010/07/nagios.png