---
title: windows 2003自带的FTP的设置？
author: admin
type: post
date: 2010-03-17T08:50:51+00:00
excerpt: |
 windows 2003自带的FTP（iis里）如何设置？打开 Internet信息服务(IIS)管理器 （如下图）



 可以看到 Internet信息服务(IIS)管理器 中已经出现了 FTP站点 菜单（如下图）



 单击 FTP站点 ，右边呈现的是 相关数据和参数（如下图）



 接着，我们来打开它的属性栏 ，右键单击它，单击属性（如下图）



 选项卡 FTP站点 下，列出来相关参数，默认的FTP的TCP连接端口是21，这个一般不改它（如下图）





 单击 安全账户 选项卡，下面可以勾选匿名，也可以添加用户账号，我们这里只是演示，所以不改它（如下图）



 接下来，单击 主目录 设置修改我们这个ftp的指向访问目录，我们这里指向 F盘（如下图）

 选择后，再单击下一步(N)> (如下图)

 设置访问权限，读取 就是只能看里面内容，能下载，但不能上传；写入，就是可以看，下载，还有上传（如下图）
url: /archives/2986
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ftp

---
windows 2003自带的FTP（iis里）如何设置？打开 **Internet信息服务(IIS)管理器** （如下图）

[![ms_ftp_1](http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_1.gif)][1]

可以看到 **Internet信息服务(IIS)管理器** 中已经出现了 **FTP站点** 菜单（如下图）

[![ms_ftp_2](http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_2.gif)][2]

单击 **FTP站点** ，右边呈现的是 相关数据和参数（如下图）

[![ms_ftp_3](http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_31.gif)][3]

接着，我们来打开它的属性栏 ，右键单击它，单击**属性**（如下图）

[![ms_ftp_4](http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_41.gif)][4]

选项卡 **FTP站点** 下，列出来相关参数，默认的FTP的TCP连接端口是21，这个一般不改它（如下图）

[![ms_ftp_5](http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_5.gif)][5]

单击 **安全账户** 选项卡，下面可以勾选匿名，也可以添加用户账号，我们这里只是演示，所以不改它（如下图）

[![ms_ftp_6](http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_6.gif)][6]

接下来，单击 **主目录** 设置修改我们这个ftp的指向访问目录，我们这里指向 F盘（如下图）

[![ms_ftp_7](http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_7.gif)][7]

选择后，再单击**下一步(N)>** (如下图)

[![ms_ftp_8](http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_8.gif)][8]

设置访问权限，读取 就是只能看里面内容，能下载，但不能上传；写入，就是可以看，下载，还有上传（如下图）

[![ms_ftp_9](http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_9.gif)][9]

在你的电脑上打开FTP客户端软件，输入IP，您windows远程桌面的用户名和密码，就可以登录FTP了。

 [1]: http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_1.gif
 [2]: http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_2.gif
 [3]: http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_31.gif
 [4]: /wp-content/uploads/2010/03/ms_ftp_41.gif
 [5]: http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_5.gif
 [6]: http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_6.gif
 [7]: http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_7.gif
 [8]: http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_8.gif
 [9]: http://blog.haohtml.com/wp-content/uploads/2010/03/ms_ftp_9.gif