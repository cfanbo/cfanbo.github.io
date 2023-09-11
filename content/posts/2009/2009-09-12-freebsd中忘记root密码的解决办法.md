---
title: FreeBSD中忘记root密码的解决办法
author: admin
type: post
date: 2009-09-12T02:49:44+00:00
excerpt: |
 重新启动FreeBSD 过往单用户更改密码
 FreeBSD 4.x 或之前的版本
 在系统启动所示以下信息时按 spacebar
 Hit [Enter] to boot immediately, or any other key for command prompt.
 Booting [kernel] in 10 seconds...
 接着在所示以下信息时输入 boot -s
 Type '?' for a list of commands, or 'help' for more detailed help.
 ok
url: /archives/2361
IM_data:
 - 'a:1:{s:50:"http://www.cublog.cn/u/184/upfile/060430122955.jpg";s:72:"http://blog.haohtml.com/wp-content/uploads/2009/09/36ee_060430122955.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 服务器

---
重新启动FreeBSD 过往单用户更改密码

FreeBSD 4.x 或之前的版本

在系统启动所示以下信息时按 spacebar

Hit [Enter] to boot immediately, or any other key for command prompt.

 Booting [kernel] in 10 seconds…接着在所示以下信息时输入 boot -s

 Type ‘?’ for a list of commands, or ‘help’ for more detailed help.

 ok


按 Enter 后系统会进行至所示以下信息



Enter full pathname of shell or RETURN for /bin/sh:

再按 Enter 进入单用户模式,所示 #

挂载档案系统,输入

# fsck -p \\文件档案检查
# mount -u / \\挂载
# mount -t ufs -a \\挂载所有文件档案

更改密码

# passwd \\更改密码

New password:_

Retype new password:_

passwd: updating the database…

passwd: done

# exit \\离开单用户进入多用户正常模式

 FreeBSD 5 或之后版本.

在系统启动所示以下界面时按 spacebar 选择 4 按 Enter 进入单用户模式

[![freebsd_get_root_password](http://blog.haohtml.com/wp-content/uploads/2009/09/freebsd_get_root_password.jpg)](http://blog.haohtml.com/wp-content/uploads/2009/09/freebsd_get_root_password.jpg)

系统会进行至所示以下信息

Enter full pathname of shell or RETURN for /bin/sh:

再按 Enter 进入单用户,所示#

挂载档案系统, 输入# fsck -p     \\文件档案检查
# mount -u /      \\ /挂载
# mount -t ufs -a     \\挂载所有文件檔案



更改密码

# passwd \\更改密码

New password:_

Retype new password:_

passwd: updating the database…

passwd: done

# exit \\离开单用户进入多用户正常模式