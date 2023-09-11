---
title: PHP.INI配置:Session配置详细说明教程
author: admin
type: post
date: 2011-04-13T04:05:54+00:00
url: /archives/9253
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - php.ini
 - session

---
网上有很多PHP.INI文件配置的中文说明，但是对于PHP初学者来说在进行PHP运行环境搭建配置时还是容易一头雾水，今天换一种角度来分享如何进行php.ini配置，以求达到解决实际问题的效果，开篇以[PHP教程][1]方式详细介绍如何通过php.ini来配置Session，以实现基本的Session应用。

我们知道在利用PHP进行购物车、用户登录等交互式网站开发时，Session是一种很好的解决方法，如果采用[XAMPP][2],[AppServ][3]等PHP安装包，一般情况下，PHP Session设置系统都会配置如果采用手动配置PHP运行环境，就需要我们通过php.ini来对Session进行配置，下面详细介绍如何进行Session配置。

**PHP运行环境说明**

我采用的是DedeAMPZ，PHP版本5.2.4，如果你是手动安装PHP运行环境，你需要将php.ini-dist或者php.ini-recommended重命名为php.ini，并将其复制在windows目录下。

**php.ini中的session配置说明**

下面介绍能让session运行的必要配置步骤

手动配置PHP运行环境时，最容易遗忘的一项是服务器端session文件的存储目录配置工作，打开php.ini文件，搜索Session，找到**session.save_path**，默认值为/tmp，代表session文件保存在c:/tmp目录下，默认tmp目录并没有创建，你可以在c盘下创建tmp目录，或者创建一个其他目录，比如leapsoulcn，再修改session.save_path的值，并去掉;，即

session.save_path = ‘/leapsoulcn’;

**注意事项**：

1、一般为了保证服务器的安全，session.save_path值最好设置为外网无法访问的目录，另外如果你是在linux服务器下进行session配置，请务必同时配置此目录为可读写权限，否则在执行session操作时会报错。

2、在使用session变量时，为了保证服务器的安全性，最好将register\_globals设置为off，以保证全局变量不混淆，在使用session\_register()注册session变量时，你可以通过系统全局变量$\_SESSION来访问，比如你注册了leapsoulcn变量，你可以通过$\_SESSION[‘leapsoulcn’]来访问此变量。 [PHP环境变量$_SERVER和系统常量详细说明](http://www.leapsoul.cn/?p=334)

**session.save_path配置其他说明事项，从php.ini配置文件翻译而来**

你可以使用”N;[MODE;]/path”这样模式定义该路径，N是一个整数，表示使用N层深度的子目录，而不是将所有数据文件都保存在一个目录下。

[MODE;]可选，必须使用8进制数，默认600(=384)，表示每个目录下最多保存的会话文件数量。[MODE;]并不会改写进程的umask。php不会自动创建这些文件夹结构。可使用ext/session目录下的mod_files.sh脚本创建。如果该文件夹可以被不安全的用户访问(比如默认的”/tmp”)，那么将会带来安全漏洞。当N>0时自动垃圾回收将会失效，具体参见下面有关垃圾搜集的部分。

如果你服务器上有多个虚拟主机，建议针对每个不同的虚拟主机分别设置各自不同的目录。

至此最基本的session配置就完成了，你只要保存php.ini，并重启apache，即可使用session功能。

**其他session配置说明**

**session.save_handler = ”files”**

默认以文件方式存取session数据，如果想要使用自定义的处理器来存取session数据，比如数据库，用”user”。

**session.use_cookies = 1**

是否使用cookies在客户端保存会话sessionid，默认为采用cookies

**session.use\_only\_cookies = 0**

是否仅仅使用cookie在客户端保存会话sessionid，这个选项可以使管理员禁止用户通过URL来传递id，默认为0，如果禁用的话，客户端如果禁用Cookie将使session无法工作。

**session.name = “PHPSESSID”**

**当做cookie name来使用的session标识名**

**session.auto_start = 0**

是否自动启动session，默认不启动，我们知道在使用session功能时，我们基本上在每个php脚本头部都会通过session\_start()函数来启动session，如果你启动这个选项，则在每个脚本头部都会自动启动session，不需要每个脚本头部都以session\_start()函数启动session，推荐关闭这个选项，采用默认值。

**session.cookie_lifetime = 0**

传递sessionid的Cookie有效期(秒)，0表示仅在浏览器打开期间有效。

**session.gc_probability = 1**

**session.gc_divisor = 100**

定义在每次初始化会话时，启动垃圾回收程序的概率。计算公式如下：session.gc\_probability/session.gc\_divisor，比如1/100，表示有1%的概率启动启动垃圾回收程序，对会话页面访问越频繁，概率就应当越小。建议值为1/1000~5000。

**session.gc_maxlifetime = 1440**

设定保存的session文件生存期，超过此参数设定秒数后，保存的数据将被视为’垃圾’并由垃圾回收程序清理。判断标准是最后访问数据的时间(对于FAT文件系统是最后刷新数据的时间)。如果多个脚本共享同一个session.save\_path目录但session.gc\_maxlifetime不同，将以所有session.gc_maxlifetime指令中的最小值为准。

如果你在session.save_path选项中设定使用子目录来存储session数据文件，垃圾回收程序不会自动启动，你必须使用自己编写的shell脚本、cron项或者其他办法来执行垃圾搜集。

比如设置”session.gc_maxlifetime=1440″ (24分钟)：

cd /path/to/sessions; find -cmin +24 | xargs rm

以上是一些常用的session配置选项说明，更多的session配置选项说明你可以参考php.ini文件中的说明。

至此，在php.ini配置文件中对session进行配置的[PHP教程][1]就介绍完毕了，通过上面的步骤实践与学习，基本的session功能都可以使用，至于session性能等其他方面则需要根据服务器环境和需求进行微调了，这个得自己体会。

**注**：[PHP网站开发教程-leapsoul.cn][4]版权所有，转载时请以链接形式注明原始出处及本声明，谢谢。



 [1]: http://www.leapsoul.cn/
 [2]: http://www.leapsoul.cn/?p=275
 [3]: http://www.leapsoul.cn/?p=292
 [4]: http://www.leapsoul.cn/ "leapsoul - 分享PHP网站开发与建设的乐趣,以PHP实例教程方式教你建站"