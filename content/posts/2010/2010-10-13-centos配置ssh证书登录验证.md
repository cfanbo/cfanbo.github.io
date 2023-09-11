---
title: CentOS配置SSH证书登录验证
author: admin
type: post
date: 2010-10-13T08:22:10+00:00
url: /archives/6024
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - ssh

---
**操作步骤：**

————————–
1）先添加一个维护账号：msa

2）然后su – msa

3）ssh-keygen -t rsa
指定密钥路径和输入口令之后，即在/home/msa/.ssh/中生成公钥和私钥：id\_rsa id\_rsa.pub

4）cat id\_rsa.pub >> authorized\_keys
至于为什么要生成这个文件，因为sshd_config里面写的就是这个。
然后chmod 400 authorized_keys，稍微保护一下。

5）用psftp把把id\_rsa拉回本地，然后把服务器上的id\_rsa和id_rsa.pub干掉

6）配置/etc/ssh/sshd_config

> Protocol 22
> ServerKeyBits 1024
> PermitRootLogin no  #禁止root登录而已，与本文无关，加上安全些

#以下三行没什么要改的，把默认的#注释去掉就行了

> RSAAuthentication yes
> PubkeyAuthentication yes
> AuthorizedKeysFile    .ssh/authorized_keys
>
> PasswordAuthentication no
> PermitEmptyPasswords no

7）重启sshd

> /sbin/service sshd restart

8）转换证书格式，迁就一下putty

运行puttygen，转换id_rsa为putty的ppk证书文件

9）配置putty登录
在connection–SSH–Auth中，点击Browse,选择刚刚转换好的证书。
然后在connection-Data填写一下auto login username，例如我的是msa
在session中填写服务器的IP地址，高兴的话可以save一下

10）解决一点小麻烦
做到这一步的时候，很可能会空欢喜一场，此时就兴冲冲的登录，没准登不进去：
No supported authentication methods available

这时可以修改一下sshd_config，把
PasswordAuthentication no临时改为：
PasswordAuthentication yes 并重启sshd

这样可以登录成功，退出登录后，再重新把PasswordAuthentication的值改为no，重启sshd，以后登录就会正常的询问你密钥文件的密码了，答对了就能高高兴兴的登进去。

至于psftp命令，加上个-i参数，指定证书文件路径就行了。

**附记：**

因为上次帮别人检查安全时，在一台Linux服务器中发现/bin下的命令很多都被病毒感染，一运行就是段错误。所以这次配置之前，先使用avscan和avast把服务器查了一遍，攘外必先安内嘛！

顺便提醒一下不小心看到这篇日记的各位，千万不要相信什么Linux服务器不怕病毒之类的鬼话！一样会中毒成为别人的肉鸡。