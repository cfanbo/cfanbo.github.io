---
title: 'CentOS下搭建Git服务器Gitosis[教程]'
author: admin
type: post
date: 2012-04-05T02:40:30+00:00
url: /archives/12658
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - centos
 - git
 - gitosis

---
说明：由于条件有限，我这里使用的是同一台centos的,但教程内容基本上通用。

**1.编译安装git**

git安装教程:

**2.安装gitosis**

 1. $ yum install python python-setuptools
 2. $ git clone [git://github.com/res0nat0r/gitosis.git](git://github.com/res0nat0r/gitosis.git)
 3. $ cd gitosis
 4. $ python setup.py install

网址： [https://github.com/res0nat0r/gitosis](https://github.com/res0nat0r/gitosis)

**3.在开发机器上生成公共密钥(用来初始化gitosis)**

 1. $ ssh-keygen -t rsa #不需要密码,一路回车就行(在本地操作)
 2. $ scp ~/.ssh/id_rsa.pub root@xxx:/tmp/ # 上传你的ssh public key到服务器

**4.初始化gitosis[服务器端]**

 1. $ adduser git # 新增一个git用户(先添加用户组　groupadd git)
 2. $ su git # 切换倒git用户下
 3. $ gitosis-init < /tmp/id\_rsa.pub # id\_rsa.pub是刚刚传过来的,注意放在/tmp目录主要是因为此目录权限所有人都有定权限的
 4. $ rm /tmp/id\_rsa.pub # id\_rsa.pub已经无用，可删除.

**5.获取并配置gitosis-admin [客户端]**

 1. $  git clone git@xxx:gitosis-admin.git  # 切换到root用户并在本地执行，获取gitosis管理项目，将会产生一个gitosis-admin的目录，里面有配置文件gitosis.conf和一个 keydir 的目录，keydir目录主要存放git用户名
 2. $  vi gitosis-admin/gitosis.conf  # 编辑gitosis-admin配置文件

如果无法git clone的话，可以使用git clone git@xxx:/home/git/repositories/gitosis-admin.git

# 在gitosis.conf底部增加

 1. [group 组名]
 2. writable = 项目名
 3. members = 用户  # 这里的用户名字 要和 keydir下的文件名字相一致

# VI下按ZZ（大写）两次会执行自动保存并退出，完成后执行

 1. $ cd gitosis-admin
 2. $ git add .
 3. $ git commit -a -m “xxx xx” # 要记住的是，如果每次添加新文件必须执行git add .，或者git add filename，如果没有新加文件，只是进行修改的话就可以执行此句。

# 修改了文件以后一定要PUSH到服务器，否则不会生效。

 1. $ git push

如果在git push的时候，遇到错误“ddress 192.168.0.77 maps to bogon, but this does not map back to the address – POSSIBLE BREAK-IN ATTEMPT!”，解决为修改/etc/hosts文件，将ip地址与主机名对应关系写进去就可以了。

**注意：** 这里我们并没有进行任何的修改的,现在只有一个管理git的项目。下面的为新添加项目的配置，大家经常用到的也就是下面的操作的。

**新建项目**

到此步就算完成gitosis的初始化了。接下来的是新建一个新项目到服务器的操作，如第5步中配置gitosis.conf文件添加的是

 1. [group project1] # 组名称
 2. writable = project1 # 项目名称
 3. members = **xxx**# 用户名xxx一定要与客户端使用的用户名完全一样，否则无权限操作

提交修改并更新到git server服务端

 1. $ git commit -a -m “添加新项目project1，新项目的目录是project1，该项目的成员是xxx“ # “”里的内容自定
 2. $ git push

将新创建的项目提交到git server 上进行登记。以便客户可以操作新项目.

对于gitosis.conf文件的配置可参考： [http://tanjunjie.blog.51cto.com/6988/967281](http://tanjunjie.blog.51cto.com/6988/967281)

# 在客户端创建项目目录 **(客户端,当前用户为 XXX )**

现在回到开发者客户端，上面创建了一个新项目project1并提交到了git server 。我们这里就创建此项目的信息.注意 项目名称 project1要与gitosis.conf文件配置一致，

 1. $ mkdir /home/用户/project1
 2. $ cd /home/用户/project1
 3. $ git init
 4. $ git add . # 新增文件 留意后面有一个点
 5. $ git commit -a -m “初始化项目project1″

# 然后就到把这个项目放到git server服务器上去.

 1. $ git remote add origin git@xxx:project1.git # xxx为服务器地址
 2. $ git push origin master

# 也可以把上面的两步合成一步

 1. $ git push git@xxx:project1.git master

说明：如果在执行 git push origin master 的时候，提示以下错误：
error: src refspec master does not match any.
error: failed to push some refs to ‘git@192.168.0.77:pro2.git’

这是由于项目为空的原因，我们在项目目录里新创建一个文件。经过->add -> commit -> push 就可以解决了

 1. $ touch a.txt
 2. $ git add a.txt
 3. $ git commit -a -m ‘add a.txt’
 4. $ git push

————————————————————————————————

如果在git clone的时候遇到“

> ## error: cannot run ssh: No such file or directory – cygwin git

”错误，则表示本机没有安装ssh命令。安装方法请参考：

有时候我们要更换电脑来重新开发项目。这个时候，只需要将id_rsa私钥放在home目录里的.ssh目录里就可以了。(有时候一个人开发多个项目，这时候可能会提示id_rsa文件已经存在。不太清楚这里如何解决？？？)

**续篇：** [git下添加新项目及用户](http://blog.haohtml.com/archives/13331)

### ====================================================

### **三、常见问题**

首先确定 /home/git/repositories/gitosis-admin.git/hooks/post-update 为可执行即属性为 0755

#### 1. git操作需要输入密码

原因
: 公密未找到

解决
: 上传id_rsa.pub到keydir并改为’gitosis帐号.pub’形式，如miao.pub。扩展名.pub不可省略

#### 2. ERROR:gitosis.serve.main:Repository read access denied

原因
: gitosis.conf中的members与keydir中的用户名不一致，如gitosis中的members = foo@bar，但keydir中的公密名却叫foo.pub

解决
: 使keydir的名称与gitosis中members所指的名称一致。
 改为members = foo 或 公密名称改为foo@bar.pub

#### 3. 相关链接

**相关文档：**