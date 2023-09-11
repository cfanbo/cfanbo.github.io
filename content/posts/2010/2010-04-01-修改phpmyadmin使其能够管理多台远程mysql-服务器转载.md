---
title: '修改phpMyAdmin使其能够管理多台远程MySQL 服务器[转载]'
author: admin
type: post
date: 2010-04-01T14:50:55+00:00
url: /archives/3200
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - mysql

---
[文章作者：张宴 本文版本：v1.2 最后修改：2007.07.09 转载请注明出处： [http://blog.s135.com](http://blog.s135.com/)]

需 求背景：
phpMyAdmin是一款不错的MySQL在线管理工具，但phpMyAdmin的cookie登录方式只能输入MySQL数据库 的用户名和密码，而想更改MySQL服务器地址和端口则须修改其配置文件config.default.php。当拥有多台数据库服务器，每台服务器又在 不同端口启动了多个MySQL服务，每次都修改配置文件就显得很麻烦，因此需要能够在登录界面直接输入MySQL服务器地址和端口的功能。

功 能要求：
假设phpMyAdmin的访问网址为 [http://192.168.1.25/phpmyadmin/](http://192.168.1.25/phpmyadmin/)，能够通过输入MySQL服务器地址、端口、 用户名、密码登录远程MySQL服务器，对远程数据库进行管理。

修改后的phpMyAdmin登录入口截图：
[![phpmyadmin2.10_index](http://blog.haohtml.com/wp-content/uploads/2010/04/phpmyadmin2.10_index.jpg)][1]

下 载地址： [http://ishare.iask.sina.com.cn/cgi-bin/fileid.cgi?fileid=1848024](http://ishare.iask.sina.com.cn/cgi-bin/fileid.cgi?fileid=1848024)

实 现步骤：

**1、打开“路径/phpmyadmin/libraries /config.default.php”，查找相关项并修改为以下内容：**

**2、打开“路径/phpmyadmin/index.php”，在文件最开头增加以下PHP代 码：**

**3、打开“路径/phpmyadmin/libraries/auth /cookie.auth.lib.php”，查找“”这行，在该行下方的第10行后（即“”这行后）增加以下HTML代码：**

图示：
[![phpmyadmin_setp3](http://blog.haohtml.com/wp-content/uploads/2010/04/phpmyadmin_setp3.jpg)](http://blog.haohtml.com/wp-content/uploads/2010/04/phpmyadmin_setp3.jpg)

**4、 创建一个可以从任何IP地址远程连接的MySQL帐号sina**

MySQL默认的帐号为root，密码为空，只允许 localhost登录，因此需要创建一个可以从任何IP地址远程连接的MySQL帐号，本例中创建的帐号为sina，密码为zhangyan。 使用该帐号从phpMyAdmin登录后，别忘了在“权限”栏中修改密码。

**(1)、Linux下的MySQL命令行客户 端添加帐号示例：**
A.登录使用默认3306端口的MySQL

引用


/usr/local/mysql/bin/mysql -u root -p


B.通过TCP连接管理不同端口的多个MySQL（注意：MySQL4.1以上版本才有 此项功能）

引用


/usr/local/mysql/bin/mysql -u root -p –protocol=tcp –host=localhost –port=3307


C.通过 socket套接字管理不同端口的多个MySQL

引 用


/usr/local/mysql/bin/mysql -u root -p –socket=/tmp/mysql3307.sock


D.通过端口和IP管理不同端口的多个MySQL

引用


/usr/local/mysql/bin/mysql -u root -p -P 3306 -h 127.0.0.1


Enter password: (输入密码，如果密码为空，直接回车)
mysql> (在这儿输入以下的语句)

引用


GRANT ALL PRIVILEGES ON *.* TO ‘sina’@’%’ IDENTIFIED BY ‘zhangyan’;


如果提示信息为Query OK, 0 rows affected (0.01 sec)，表示执行成功。

**(2)、Windows下的MySQL命令行客户端添加帐号示 例：**
A.管理使用默认3306端口的MySQL

引用


d:\apmserv\mysql\bin\mysql.exe -u root -p


B.管理不同端口的多个MySQL

引用


d:\apmserv\mysql\bin\mysql.exe -u root -p –port=3307


Enter password: (输入密码，如果密码为空，直接回车)
mysql> (在这儿输入以下的语句)

引用


GRANT ALL PRIVILEGES ON *.* TO sina@”%”;


如果提示信息为Query OK, 0 rows affected (0.01 sec)，表示执行成功。

 [1]: http://blog.haohtml.com/wp-content/uploads/2010/04/phpmyadmin2.10_index.jpg