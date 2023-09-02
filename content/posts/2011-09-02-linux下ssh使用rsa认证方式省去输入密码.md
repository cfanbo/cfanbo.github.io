---
title: 'linux下ssh使用rsa认证教程[原创]'
author: admin
type: post
date: 2011-09-02T16:03:42+00:00
url: /archives/11214
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - rsa
 - ssh
 - sshd

---
下面我们来对linux(centos)平台如何使用ssh的rsa认证功能来实现安全登录服务器的教程.为了安全我们一般不直接使用root这个用户,而使用其它用户来代替.如果需要root权限的时候,直接在服务器上进行su命令进行用户切换就可以了.

**一.配置/etc/ssh/ssh_config文件**

>

```
ServerKeyBits 1024 //# 注释取消，将768改为1024

PermitRootLogin no //# 注释取消，将yes改为no 禁止root登录
```

>
>

```
RSAAuthentication yes  //# 启用 RSA 认证
PubkeyAuthentication yes     //# 启用公钥认证
AuthorizedKeysFile     //# .ssh/authorized_keys   # 验证公钥的存放路径
```

>
>

```
PermitEmptyPasswords no   //# 取消注释，禁止空密码登录
PasswordAuthentication no //# 取消注释，禁止使用密码方式登录，有密钥谁还用密码啊
注意一下，在centos5.0之前SSH服务需要指明版本，#Protocol 2,1 把前面的注释取消，选择自己需要的版本就行了。
```

重启sshd服务

> service sshd restart**说明:**

如果想做到最大化安全链接，可以考虑在配置有双网卡的服务器上设置只允许内网链接SSH，方法很简单，
首先在**/etc/hosts.deny**文件最后一行添加一句sshd: ALL
然后在**/etc/hosts.allow**的最后一行加上一句sshd: 192.168.0.
然后保存退出。

**二.生成密匙**

使用Linux主机生成的密匙,这里使用的是sysadmin这个用户,如果是使用其它用户的话,请在相应用户的home目录下面的.ssh文件夹里进行操作.这里我们创建一个sysadmin用户.

> #useradd sysadmin -g wheel

#passwd sysadmin

#su sysadmin**1、生成密匙**

```
[sysadmin@localhost ssh]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/sysadmin/.ssh/id_rsa):
Create directory '/home/sysadmin/.ssh'.
Enter passphrase (empty for no passphrase):      #//在这里输入密钥的密码
Enter same passphrase again:                     #//再次输入密码确认
Your identification has been saved in /home/sysadmin/.ssh/id_rsa.
Your public key has been saved in /home/sysadmin/.ssh/id_rsa.pub.
The key fingerprint is:
4a:51:5e:8b:67:0a:e9:9f:b5:b6:c9:f0:94:43:38:e9 sysadmin@localhost.localdomain
The key's randomart image is:
+--[ RSA 2048]----+
|        . .      |
|       + o .     |
|      + o +      |
|     . o *       |
|      o S o      |
|     . + = o     |
|      . E *      |
|         * +     |
|          =      |
+-----------------+
```

**2、将 /sysadmin/.ssh/id_rsa.pub改名为/sysadmin/.ssh/authorized_keys**

```
#切换到sysadmin的home目录下的.ssh目录里.
[sysadmin@localhost .ssh]$ cd ~/.ssh
[sysadmin@localhost .ssh]$ ls -l
total 8
-rw-------. 1 sysadmin wheel 1743 Sep  3 09:58 id_rsa
-rw-r--r--. 1 sysadmin wheel  412 Sep  3 09:58 id_rsa.pub
```

```
[sysadmin@localhost .ssh]$ mv id_rsa.pub authorized_keys
[sysadmin@localhost .ssh]$ ls -l
total 8
-rw-r--r--. 1 sysadmin wheel  412 Sep  3 09:58 authorized_keys
-rw-------. 1 sysadmin wheel 1743 Sep  3 09:58 id_rsa
```

```
[sysadmin@localhost .ssh]$ chmod 400 authorized_keys
[sysadmin@localhost .ssh]$ ls -l
total 8
-r--------. 1 sysadmin wheel  412 Sep  3 09:58 authorized_keys
-rw-------. 1 sysadmin wheel 1743 Sep  3 09:58 id_rsa
```

**3、将私钥id_rsa拷贝到远程客户端**将id_rsa文件存放在U盘上或者其它地方.以便随时可以使用.

1)、如果远程客户端是linux，拷贝到远程客户端/root/.ssh/即可

 2)、putty作为远程客户端由于putty不能识别直接从服务器拷贝来的私钥，需要使用puttygen.exe进行格式转换

 (1)、打开puttygen.exe –> Conversions –> Import Key

[![](http://blog.haohtml.com/wp-content/uploads/2011/09/putty_key_generator_input_password.jpg)][1]

然后输入在服务器上生成密钥的时候的设置的密码

[![](http://blog.haohtml.com/wp-content/uploads/2011/09/putty_key_generator.jpg)](http://blog.haohtml.com/wp-content/uploads/2011/09/putty_key_generator.jpg)

 (2)、选择拷贝过来的私钥文件id_rsa

 (3)、Save private key->id_rsa.ppk(也可以修改为其它名字,这里用了haohtml_ssh,保存私钥)[![](http://blog.haohtml.com/wp-content/uploads/2011/09/putty_private_key.jpg)](http://blog.haohtml.com/wp-content/uploads/2011/09/putty_private_key.jpg)

**4、打开putty.exe**

 1)、Session –> Host Name (填写服务器地址或者域名)[![](http://blog.haohtml.com/wp-content/uploads/2011/09/putty_ssh.jpg)](http://blog.haohtml.com/wp-content/uploads/2011/09/putty_ssh.jpg)

 2)、Connection –> SSH –> Auth (点Browse选择刚生成的haohtml_ssh.ppk)[![](http://blog.haohtml.com/wp-content/uploads/2011/09/putty_ssh_select_private_key.jpg)](http://blog.haohtml.com/wp-content/uploads/2011/09/putty_ssh_select_private_key.jpg)

 3)、open

 成功打开后出现如下提示：

 login as: sysadmin #//这里输入sysadmin用户

Authenticating with public key “imported-openssh-key”

然后输入在服务器上生成密钥的时候设置的密码就可以了.

[![](http://blog.haohtml.com/wp-content/uploads/2011/09/putty-ssh-rsa-success.jpg)][2]

puttygen.exe和putty.exe文件下载地址见:

转载请注明本文来自:

 [1]: http://blog.haohtml.com/wp-content/uploads/2011/09/putty_key_generator_input_password.jpg
 [2]: http://blog.haohtml.com/wp-content/uploads/2011/09/putty-ssh-rsa-success.jpg