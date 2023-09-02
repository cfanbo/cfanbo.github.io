---
title: linux下varnish配置及使用教程
author: admin
type: post
date: 2011-11-18T03:11:49+00:00
url: /archives/11985
IM_contentdowned:
 - 1
categories:
 - 系统架构
 - 服务器
tags:
 - varnish

---
centos6.0  32位
Varnish3.0.2

我们先配置nginx环境.参考教程:http://blog.haohtml.com/archives/6051
并修改nginx的监听端口为81.下面我们varnish监听的端口为80端口.

==============================================================
目前varnish的最新版本为3.0.2,这里我们使用最新的稳定版本

```
cd /usr/local/soft
wget http://repo.varnish-cache.org/source/varnish-3.0.2.tar.gz
tar zxvf varnish-3.0.2.tar.gz
cd varnish-3.0.2
./configure --prefix=/usr/local/varnish
make && make install
```

如果在执行./configure命令的过程中遇到”No package ‘libpcre’ found”的错误提示信息的话,需要执行以下命令

>

```
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
```

```
即可(这个已经在前面配置nginx环境的时候,安装过了pcre库了,这里是环境变量的问题).
```

现在我们确认一下是否安装成功.

[root@bogon varnish-3.0.2]# ls /usr/local/varnish
bin etc include lib sbin share var

其中sbin目录里为varnish主程序文件,bin目录里为varnish的管理命令,etc目录为配置文件,share目录为手册,lib目录为一些库文件,include目录为一些程序所需的c语言里的.h头文件

**varnish语法**

```
[root@bogon varnish-3.0.2]#cd /usr/local/varnish/sbin
[root@bogon sbin]# ./varnishd --help
./varnishd: invalid option -- '-'
usage: varnishd [options]
    -a address:port              # HTTP listen address and port
    -b address:port              # backend address and port
                                 #    -b <hostname_or_IP>
                                 #    -b '<hostname_or_IP>:<port_or_service>'
    -C                           # print VCL code compiled to C language
    -d                           # debug
    -f file                      # VCL script
    -F                           # Run in foreground
    -h kind[,hashoptions]        # Hash specification
                                 #   -h critbit [default]
                                 #   -h simple_list
                                 #   -h classic
                                 #   -h classic,<buckets>
    -i identity                  # Identity of varnish instance
    -l shl,free,fill             # Size of shared memory file
                                 #   shl: space for SHL records [80m]
                                 #   free: space for other allocations [1m]
                                 #   fill: prefill new file [+]
    -M address:port              # Reverse CLI destination.
    -n dir                       # varnishd working directory
    -P file                      # PID file
    -p param=value               # set parameter
    -s kind[,storageoptions]     # Backend storage specification
                                 #   -s malloc
                                 #   -s file  [default: use /tmp]
                                 #   -s file,<dir_or_file>
                                 #   -s file,<dir_or_file>,<size>
                                 #   -s persist{experimenta}
                                 #   -s file,<dir_or_file>,<size>,<granularity>
    -t                           # Default TTL
    -S secret-file               # Secret file for CLI authentication
    -T address:port              # Telnet listen address and port
    -V                           # version
    -w int[,int[,int]]           # Number of worker threads
                                 #   -w <fixed_count>
                                 #   -w min,max
                                 #   -w min,max,timeout [default: -w2,500,300]
    -u user                      # Priviledge separation user id
```

**参数详解:**

> ****-u 以什么用户运行
> -g 用户组
> -f varnish配置文件
> -a 绑定ip和端口(前端监听端口,用户请求时,与参数-b指定的Backend机器通讯)
> -b 后端监听端口(为了安全,这里一般为局域网的机器ip段)
> -n 指定一个实例名
> -T varnish管理端口,主要用来清除缓存
> -s varnish缓存文件位置及大小(size默认的单位是bytes,可用的有K,M,G,T.默认没有限制)

**运行varnish**

```
mkdir /var/vcache
chown -R www:www /var/vcache
./varnishd -a 0.0.0.0:80 -b 127.0.0.1:81 -s malloc,1G -w2,500,300 -u www -g www -T 127.0.0.1:2000
```

注意:用户www:www我们上面在配置nginx的时候已经创建过了.
-a 这一句的意思是制定 varnish 监听所有 IP 发给 8080 端口的 http 请求，如果在生产环境下，您应该让varnish监听80，这也是默认的。
-b 表示监听本机的81端口(表示后面的nginx,不指定这个参数的话,需要修改配置文件default.vcl并,全同时使用-f 指定配置文件,一般为内网的机器ip地址,这样比较的安全)
–s 选项用来确定 varnish 使用的存储类型和存储容量，我使用的是 malloc 类型（malloc 是一个 C 函数，用于分配内存空间），  1G  定义多少内存被 malloced，1G = 1gigabyte。也可以使用文件形式存储,我在下面的vmware里用file的时候,会提示以下信息

```
WARNING: (-sfile) file size reduced to 4844968345 (80% of available disk space)
NB: Storage size limited to 2GB on 32 bit architecture,
NB: otherwise we could run out of address space.)
```

-T 127.0.0.1:2000 Varnish有一个基于文本的管理接口，启动它的话可以在不停止 varnish 的情况下来管理 varnish。您可以指定管理软件监听哪个接口。当然您不能让全世界的人都能访问您的varnish管理接口，因为他们可以很轻松的通过访问 varnish管理接口来获得您的root访问权限。我推荐只让它监听本机端口。如果您的系统里有您不完全信任的用户，您可以通过防火墙规则来限制他访问varnish 的管理端口。

配置开机自动启动Varnish

>

> vi /etc/rc.local
>

在末尾增加以下内容：

>

> ulimit -SHn 51200

/usr/local/varnish/sbin/varnishd -f /usr/local/varnish/vcl.conf -a 0.0.0.0:80 -s file,/var/vcache/varnish_cache.data,1G -g www -u www -w 30000,51200,10 -T 127.0.0.1:3500

/usr/local/varnish/bin/varnishncsa -n /var/vcache -w /var/log/varnish.log &
>

日志目录/var/vcache对于www:www用户要有操作权限.

如果没有指定-f参数,则使用varnish的默认配置文件/usr/local/varnish/etc/varnish/default.vcl

发现在给varnishd命令指定-n /var/vcache参数后,用 [varnishlog](http://blog.haohtml.com/archives/12014) 命令检测的时候,无法检查任何信息的.怀疑这里有问题.

**检查varnish是否正常**

这时我们可以通过浏览器里输入http://192.168.0.71,如果可能看到”Welcome to nginx!”字样则表示成功(视每个人的环境而定,只要保证和81端口看到的信息一样就可以了).来查看是否可以正常访问网站.

这里我用curl来检查文件头

>

```
[root@bogon lib]# curl -I http://192.168.0.71/
HTTP/1.1 200 OK
Server: nginx/1.1.1
Content-Type: text/html
Last-Modified: Thu, 29 Sep 2011 16:35:43 GMT
Content-Length: 151
Accept-Ranges: bytes
Date: Fri, 18 Nov 2011 03:31:23 GMT
X-Varnish: 942185170 942185168
Age: 10
Via: 1.1 varnish
Connection: keep-alive
```

可以看到红色的字,表示从varnish里读取了.

使用 varnish 后，web 应用程序是否加速，取决于一些原因。如果您的程序的每个会都使用cookies或者您每个程序都需要三次握手认证，这样varnish就不能缓存更多的数据我们暂时忽略这个问题，等到“提高缓存命中率”这节的时候我们再继续讨论这个问题。想要知道varnish对您的网站做了什么，请查看 logs。

**停止varnish**

> #pkill varnishd

注意:

Varnish与Squid不同,varnish默认会缓存所有的文件,包括动态的php程序文件,而squid默认只会缓存一些静态文件,动态的文件不会缓存的.

[点击下载Varnish权威指南-中文版.pdf](http://docs.haohtml.com/download/cdn/Varnish%c8%a8%cd%fe%d6%b8%c4%cf-%d6%d0%ce%c4%b0%e6.pdf  "Varnish权威指南-中文版")

**更多用法请参考:**