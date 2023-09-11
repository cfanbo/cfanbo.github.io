---
title: Cannot access the Hardware Clock via any known method.的解决办法
author: admin
type: post
date: 2011-07-21T13:46:05+00:00
url: /archives/10556
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - dzselect
 - hwclock

---
今天在服务器上修改时间的进修，发现用date -s 21:45:12不起作用，提示＂

```
Cannot access the Hardware Clock via any known method.
Use the --debug option to see the details of our search for an access method.
```

＂错误，后来google了一下，有人说在64位平台的原因，说是一个bug的．

在执行clock -w 和hwclock命令的时候，总提示错误信息．这里介绍一种方法：

#tzselect

然后选择”5) Asia”,回车，选择国家＂ 9) China＂回车，在选择的地区里选择＂1) east China – Beijing, Guangdong, Shanghai, etc.＂,最后选择＂1) Yes＂对上面的设置进行确认即可．会提示以下信息，这时时间已经正常了．为了长久有效，可以添加到.profile文件里，我是添加到/etc/profile文件里了，不知道对否的．反正时间是过来了．

```
You can make this change permanent for yourself by appending the line
        TZ='Asia/Shanghai'; export TZ
to the file '.profile' in your home directory; then log out and log in again.

Here is that TZ value again, this time on standard output so that you
can use the /usr/bin/tzselect command in shell scripts:
Asia/Shanghai
```