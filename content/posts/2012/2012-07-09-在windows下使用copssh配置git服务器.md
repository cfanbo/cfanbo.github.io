---
title: 在WINDOWS下使用copSSH配置GIT服务器
author: admin
type: post
date: 2012-07-09T04:38:37+00:00
url: /archives/13128
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - git

---
近日对GIT进行了研究，发现还真是个好东东，但是在GIT服务器的配置上，在试用了多个SSH服务器之后，始终未能搞定，导致几近崩溃；最终靠着秉承“外事问谷歌，内事问百度”的理念，终于找到了一篇E文的博客，才算搞定。今把过程展示出来，希望对大家能有帮助。（注：本文严重参考了以下博客 [http://www.timdavis.com.au/git/setting-up-a-msysgit-server-with-copssh-on-windows/](http://www.timdavis.com.au/git/setting-up-a-msysgit-server-with-copssh-on-windows/)，在此表示强烈感谢）
**基本原理：**使用copSSH在WINDOWS（XP）上建立SSH服务器；使用生成的“公钥-私钥”对作为身份标识；在服务器上配置SHELL脚本环境；配置客户端，加载私钥。详细过程如下：
**安装前准备：**
Download [copSSH](http://www.itefix.no/copssh/) [ [SourceForge Link](http://sourceforge.net/projects/sereds/files/Copssh/)] （注：SSH服务器软件）


Download [msysgit](http://code.google.com/p/msysgit/) （注：WINDOWS下的git安装包）
Download [TortiseGIT](http://code.google.com/p/tortoisegit/) （注：WINDOWS下的git图形化软件，与TortiseSVN是同门）
Download [PuTTY Installer](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) （注：生成公钥-私钥对的软件，并可用于SSH客户端的登陆）
**Step1 －安装copSSH**
1.将copSSH（basic edition 2.0.0）安装到路径 c:\SSH
2.安装过程中写下SvcCOPSSH的密码，你可能永远不会用到，但写下也无伤大雅。
3.启动copSSH，选择 开始->所有程序->copSSH->control panel；然后激活一个用户（假定为Administrator，选择Users->Add，下一步，选择一个用户，不要勾选Allow password authenticatin选项，点击forward，OK。
4.其他关于public keys的事情无需操作，后面还会讲到。
**Step2－配置copSSH**
1.选择路径－C:\SSH\etc ，在记事本中打开ssh\_config 和sshd\_config.(注意：两个文件有一个字母“d”的区别）
2.ssh_config －删除Port前的#号，设置端口号，这里采用默认端口22
3.sshd_config －保证端口号一致
4.确定系统防火墙中该端口未关闭。（这一点很重要）
5.重启系统
**Step3－安装Putty**
1.重启之后，继续回来，现在可以安装Putty Installer了。
2.导航至你的安装路径，通常为c:\program files\Putty
3.打开PuttyGen.exe
4.选择生成密钥的长度4096
5.在空白面板处不停地晃动鼠标（用于生成随机种子），直到生成结束。（不要关闭PuttyGen）
6.来到路径c:\SSH\Home\Administrator\.ssh\ （这个路径在你使用copSSH激活用户时会产生，根据你激活的用户名，选择相应的路径），创建文件authorized_keys （注意没有后缀名）
7.打开PuttyGen，复制Public Key（公钥）到文件authorized_keys ，并保存
8.在PuttyGen中，将Private Key（私钥）保存为private\_key.ppk，保存在同一目录下。－比如，我的保存目录为c:\SSH\Home\Administrator\.ssh\private\_key.ppk
9.现在目录下应该有两个文件了，authorized\_keys 和 private\_key.ppk
10.为了测试连接，运行putty.exe
11.在打开的界面中输入IP 地址（本机可以为localhost）和端口号
12.打开左侧的菜单，选择Connection-SSH-Auth，选择你的私钥文件，c:\SSH\Home\\.ssh\private_key.ppk
13.点击Open，就会打开终端，让你输入Login Name，输入Administrator（注意大小写）
14.你会看到显示接受你的公钥（Accept Public Key），客户端登陆成功，登陆信息也会缓存起来。
**Step 4－安装 msysgit和TortiseGIT**
1.安装msysgit的过程中一路下一步即可，假定你的路径为C:\msysgit
2.安装TortiseGIT，完成之后
1）在任意路径点右键，选择TortiseGIT-Settings，设置git.exe的路径为c:\msysgit\msysgit\bin，即为msysgit的安装路径
2）在左侧菜单中选择Network，选择SSH Client为putty中的plink.exe（如我的路径为C:\Program Files\PuTTY\plink.exe）
3.将几个GIT运行中需要的文件复制到SSH服务器目录，当客户端远程登陆上来以后需要执行这些文件，文件源路径为c:\msysgit\msysgit\Git\libexec\git-core ，要复制的文件包括

> git.exe
> git-receive-pack.exe
> git-upload-archive.exe
> and git-upload-pack.exe

将以上文件复制到 C:\SSH\Bin

将$Git\bin目录下的libiconv-2.dll复制到$ICW\bin目录下

**Step5－配置用户环境**
1.对于copSSH来说，其默认的$ HOME环境为c:\Documents and Settings\，GIT也将会在该目录下寻找authorized_keys 文件。当然，这是咱要避免的事儿，我们要将GIT的路径重定向到C:\SSH\Home\\.ssh 。
2.选择路径C:\SSH\Home\Administrator\，打开.bashrc文件，在# User dependent .bashrc file下面加上这样一段：export HOME=/c/SSH/home/Administrator Shell Options,（注意不要有其他空格出现），然后选择保存。
3.把该文件复制到用户目录下，如： c:\Documents and Settings\Administrator\
**Step7－使用GIT和Plink**
1.打开路径C:\SSH\home\Administrator，创建文件夹myapp.git
2. 在该文件夹上点右键，选择git create repository here,勾选make it bare，服务器文件仓库创建成功。
3.导航至路径c:\Program Files\PuTTY ，打开pageant.exe，选择add key，将你的私钥（private_key.ppk）加载上。
4.然后右键选择 git clone，url设为ssh://administrator@127.0.0.1:22/SSH/Home/administrator/myapp.git ，如果clone成功，恭喜你，大功告成！
关于git的操作详见git的使用说明，这里推荐Pro Git 简体中文版，翻译的很不错。

注：在执行git clone时可能会报错（该错误在所参考的E文中未提及，把俺害得不轻），如果是关于某个dll文件的错（具体是哪个文件记不清了，遇到的朋友可以根据文件名，在msysigt目录下搜索即可找到），可以将该文件同样复制到C:\SSH\Bin下，然后就可以正常运行了。