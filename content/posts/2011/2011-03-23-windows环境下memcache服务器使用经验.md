---
title: windows环境下memcache服务器使用经验
author: admin
type: post
date: 2011-03-23T06:06:42+00:00
url: /archives/8098
IM_contentdowned:
 - 1
categories:
 - 程序开发
 - 服务器
tags:
 - memcache

---
将memcache服务器安装包解压到C:\memcached文件夹后，使用cmd命令窗口安装。

1>开始>运行:CMD(确定)

2>cd C:\memcached（回车）

3>memcached -d install(回车 这步执行安装)

4>memcached -d start(回车 这步执行启动memcache服务器，默认分配64M内存，使用11211端口)

此时memcache服务器已经可以正常使用了。

**memcache服务器安全：**

Memcache服务器端都是直接通过客户端连接后直接操作，没有任何的验证过程，这样如果服务器是直接暴露在互联网上的话是比较危险，轻则数据泄 露被其他无关人员查看，重则服务器被入侵，况且里面可能存在一些我们未知的bug或者是缓冲区溢出的情况，这些都是我们未知的，所以危险性是可以预见的。 为了安全起见，做两点建议，能够稍微的防止黑客的入侵或者数据的泄露。

现在就关于修改memcache服务器配置的问题说明如下：

1>用内网ip的方式提供web应用服务器调用，不允许直接通过外网调用，如将memcache服务器放在192.168.1.55的服务器上

2>修改端口，如改为11200

3>分配内存，如分配1024M(1G内存)

**方法如下：**

1>开始>运行:CMD(确定)

2>cd C:\memcached（回车）

3>memcached -m 1024 -p 11200 -l 192.168.1.55(回车)

注意，此时命令行不会回到C:\memcached>状态，并且实际上memcache服务器悄悄变为stop状态了。此窗口不可以关闭。新开一个cmd窗口

4>开始>运行:CMD(确定)

5>cd C:\memcached（回车）

6>memcached -d start(回车)可以关闭此cmd窗口。

此时可以使用新配置的memcache服务器了。



上述方法虽然解决了修改默认配置的问题，但是始终会有一个cmd窗口不可以关闭，否则就回到11211端口的默认配置。

更好的解决方案是通过修改服务的注册表配置：

1>开始>运行：regedit(回车)

2>在注册表中找到：HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Services\memcached Server

3>默认的ImagePath键的值是：”c:\memcached\memcached.exe” -d runservice，改为：”c:\memcached\memcached.exe” -d runservice -m 512 -p  11200 -l 192.168.1.55（确定，关闭注册表）

4>我的电脑（右键）>管理>服务 找到memcache的服务，重新启动一次即可生效。

此时，同网段内的电脑仍然可以利用这台memcache服务器，我们限定指定的web应用服务器才能够使用，通过防火墙的方式。如只允许 192.168.1.2这台Web服务器对Memcache服务器的访问，能够有效的阻止一些非法访问，相应的也可以增加一些其他的规则来加强安全性，这 个可以根据自己的需要来做。