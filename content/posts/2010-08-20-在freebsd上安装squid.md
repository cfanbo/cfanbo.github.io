---
title: 在FreeBSD上安装Squid
author: admin
type: post
date: 2010-08-20T07:53:41+00:00
url: /archives/5178
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Squid

---

**Squid 2.5下载 & 安装**

squid的2.5最新版本是squid-2.5.STABLE7,先下载安装包,再安装:

```
 #cd /tmp
 #mkdir squidinstall
 #cd squidinstall
 #fetch http://www.squid-cache.org/Versions/v2/2.5/squid-2.5.STABLE7.tar.gz
 #tar xzvf squid-2.5.STABLE7.tar.gz
 #cd squid-2.5.STABLE7
 #./configure –prefix=/usr/local/squid
 #make
 #make install
```



看到类似于下图的提示,并且没有出现 Error Code :1 之类的错误提示,证明Squid已经安装完成了!

*********************************

/usr/local/squid/sbin/squid -z

****************************

/usr/local/squid/sbin/squid

************************

**配置Squid.conf**

好,接下来要做的仅仅是配置Squid.conf.

>

> #ee /usr/local/squid/etc/squid.conf
>

>
>

> 但是原来的squid.conf并不是最好的,最好是你自己新建一个Squid.conf.
>

>
>

> #cd /usr/local/squid/etc
>

>
>

> #mv squid.conf squid.conf.bak
>

>
>

> #touch squid.conf
>

>
>

> #ee squid.conf
>

照我的Squid.conf写,就能正常运行啦. 在ee编辑器中输入:

>

> http_port 3128 # squid 的端口
>

>
>

> cache_dir ufs /cache 530 16 256
>

>
>

> 缓存目录:/cache 类型:ufs 大小:530mb 允许Squid在目录下建立一级(16)和二级目录(256)

>
>

> cache_mem 32 MB # cache内存大小:32mb
>

>
>

> cache_store_log /var/log/squid/store.log #Squid的日志1:在/var/log/squid下
>

>
>

> cache_access_log /var/log/squid/access.log #Squid的日志2:在/var/log/squid下
>

>
>

> cache_log /var/log/squid/cache.log #Squid的日志3:在/var/log/squid下
>

>
>

> ### cache user
>

>
>

> cache_effective_user nobody #缓存用户UID
>

>
>

> cache_effective_group nogroup #缓存用户组 GID
>

>
>

> ### cache admin
>

>
>

> visible_hostname etclub.3322.org #发生错误时,生成提示所显示的缓存服务器名
>

>
>

> cache_mgr horus@etclub.3322.org #发生错误时,生成提示所显示的缓存服务器管理员名
>

>
>

> acl badurls dstdomain popme.163.com
>

>
>

> http_access deny badurls
>

>
>

> #以上2句不允许使用该缓存服务器访问popme.163.com
>

>
>

> acl badwords url_regex -i sex
>

>
>

> http_access deny badwords
>

>
>

> #以上2句不允许使用该缓存服务器访问URL正则表达式中含sex字样的URL
>

>
>

> httpd_accel_host virtual
>

>
>

> httpd_accel_port 80
>

>
>

> httpd_accel_with_proxy on
>

>
>

> httpd_accel_uses_host_header on
>

>
>

> #httpd 透明代理设置
>

>
>

> acl all src 0.0.0.0/0.0.0.0
>

>
>

> http_access allow all
>

>
>

> #以上2句允许所有ip使用该缓存服务器,这两句要放在所有的ACL语句的最后!
>

按Ctrl + C 在command:后输入exit,再回车.存盘退出. 以上是一个简单,但足以正常工作的squid.conf. 接下来,建立缓存目录和Squid的日志.

**建立Squid的日志&缓存目录**

>

> #mkdir /squid
>

>
>

> #chmod 777 /squid (缓存目录必须可写!)
>

>
>

> #chown -R nobody:nogroup /squid
>

>
>

> #cd /var/log
>

>
>

> #mkdir squid
>

>
>

> #cd squid
>

>
>

> #touch access.log
>

>
>

> #touch cache.log
>

>
>

> #touch store.log
>

>
>

> #cd ..
>

>
>

> #chown -R nobody:nogroup /var/log/squid
>

>
>

> #chown -R nobody:nogroup /usr/local/squid
>

然后:你应该让squid在/squid建立缓存文件系统

>

> #/usr/local/squid/sbin/squid -z
>

squid提示:Creating swap … 然后回到shell提示符:#. 注意:以上指不出意外的话,若出现visible_hostname错误的话,证明你的squid.conf没写完整.

[![](https://blogstatic.haohtml.com//uploads/2023/09/issue16_01.jpg)](http://blog.haohtml.com/wp-content/uploads/2010/08/issue16_01.jpg)

**运行Squid**

好了,运行你的Squid吧!

>

> #/usr/local/squid/sbin/squid
>

可以用

>

> #squid -CDNd1
>

查看squid服务是否正常

**Q & A**

Q:如何判断已经成功实现缓存功能

A:

#ps -waux | grep squid

#cat /var/log/squid/cache.log (有没有正常输出)

#netstat -a | more (找3128端口)

在IE里设置好缓存服务器的地址\端口.乱打一个URL,如:

www.gfnjigj.fdg

看到:

您所请求的网址（URL）无法获取

——————————————————————————–

当尝试读取以下网址（URL）时： http://www.gfnjigj.fdg/

发生了下列的错误：

无法将您输入的主机名称：www.gfnjigj.fdg转换成 IP 地址 域名服务器返回以下讯息：

Name Error: The domain name does not exist. 这表示：

The cache was not able to resolve the hostname presented in the URL. Check if the address is correct. 缓存服务器无法解析您输入网址（URL）中的主机名称，请检查该名称是否正确。

本缓存服务器管理员：horus@etclub.3322.org

——————————————————————————–

Generated Sun, 11 Jul 2004 06:00:58 GMT by etclub.3322.org (squid/2.5.STABLE5) 就成功啦!!! 结果,访问www.163.com,就…

[![](https://blogstatic.haohtml.com//uploads/2023/09/issue16_02.jpg)](http://blog.haohtml.com/wp-content/uploads/2010/08/issue16_02.jpg)

**重要Tip**

小技巧:安装好了之后,也许错误提示是英文的,这时候,你只要把/usr/local/squid/share/errors下的English目录和Simplify_Chinese目录互换名字,错误提示就成了中文啦(骗一下Squid…)呵呵

浅谈Squid的ACL语法

ACL,Access Control List,访问控制列表.它的语法是: (在/usr/local/squid/etc/squid.conf里添加)

acl 表名 表类型 [-i] 表的值

http_access [allow/deny] 表名下面分条解释:

表名:可以自定义

表类型:表类型有

src 源地址:客户机的IP地址

dst 目的地址:服务器的IP地址

srcdomain 源域:客户机所属的域

dstdomain 目的域:服务器所属的域

url_regex URL正则表达式(字符串部分)

urlpath_regex URL正则表达式中的路径

time [星期] [时间段]

maxconn 客户端的最大连接数

 -i 这个参数使Squid不区分大小写

表的值:随表的类型不同而不同

注意:time中的星期要用如下字符:

S (Sunday,星期日) M(Monday,星期一) T(Tuesday,星期二) W(Wednesday,星期三)

H(Thursday,星期四) F(Friday,星期五) A(Saturday,星期六)

时间段的表示方式是: XX:00-YY:00 如: 20:00-22:00

http_access 选项允许你设置一个表是允许(allow)还是拒绝(deny)

下面举几个例子: 防3721的ACL.在squid.conf中加入:

acl badurls dstdomain -i www.3721.com www.3721.net download.3721.com cnsmin.3721.com

http_access deny badurls

acl badkeywords url_regex -i 3721.com 3721.net

http_access deny badkeywords

解释:

badurls 和 badkeywords 是你自定义的表名.

dstdomain 是服务器的域名(目的域) 而 url_regex 则是URL正则表达式(字符串部分)包含的内容.

http_access 选项的 deny 则是把表badurls和表badkeywords的访问拒绝.

禁止下载Flash:

acl badfiles urlpath_regex -i \.swf$

http_access deny badfiles

大家在今后的配置中,慢慢体会ACL的用法吧! 达到目的喽:

错误

您所请求的网址（URL）无法获取

——————————————————————————–

当尝试读取以下网址（URL）时： http://www.3721.com/ 发生了下列的错误：

Access Denied. 拒绝访问

Access control configuration prevents your request from being allowed at this time. Please contact your service provider if you feel this is incorrect. 当前的存取控制设定禁止您的请求被接受，如果您觉得这是错误的，请与您网路服务的提供者联系。本缓存服务器管理员：horus@etclub.3322.org

——————————————————————————–

Generated Wed, 07 Jul 2004 06:32:41 GMT by etclub.3322.org (squid/2.5.STABLE5)

后记

仅以此Howto献给和我一样用VM玩FreeBSD的小鸟们. 这篇Howto总算是完成了,做了一些修改. 谈到了Squid安装,Squid的ACL语法等. 希望高手别嫌弃这篇Howto浅显,菜鸟晕哥别嫌弃这篇Howto太长&难懂,毕竟这是我第一次写Howto,第一次写这么长的Howto,请大家包涵 . 最后:真切祝愿大家能顺利完成FreeBSD+Squid !

(http://www.fanqiang.com)

原文链接： [http://journal.cnfug.org/issue16/000092.html](http://journal.cnfug.org/issue16/000092.html)