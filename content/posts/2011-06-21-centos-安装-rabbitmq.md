---
title: centos下安装RabbitMQ消息队列
author: admin
type: post
date: 2011-06-21T02:30:57+00:00
url: /archives/9901
IM_contentdowned:
 - 1
categories:
 - 系统架构
 - 服务器
tags:
 - centos
 - RabbitMQ
 - 消息队列

---
这里环境为centos7 64位.
一。安装erlang

[shell]su -c ‘rpm -Uvh http://download.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm’
sudo yum install erlang[/shell]

二。安装rabbitmq

我们是用CentOS7（RHEL7也一样），可以从这里： [http://fedoraproject.org/wiki/EPEL/FAQ#howtouse](http://fedoraproject.org/wiki/EPEL/FAQ#howtouse) 找到安装有erlang的RHEL7（CentOS同）软件仓库并安装：

[shell]
wget -c http://www.rabbitmq.com/releases/rabbitmq-server/v3.5.0/rabbitmq-server-3.5.0-1.noarch.rpm

sudo rpm –import http://www.rabbitmq.com/rabbitmq-signing-key-public.asc
sudo yum install rabbitmq-server-3.5.0-1.noarch.rpm
[/shell]

三。启用rabbitmq

[shell]sudo chkconfig rabbitmq-server on[/shell]

As an administrator, start and stop the server as usual using /sbin/service rabbitmq-server stop/start/etc.

[shell]sudo /sbin/service rabbitmq-server start[/shell]

注意：如果通过上面的start命令启动失败，就检查一下下面的端口是否被占用，否则服务启动不了：

 * 4369（epmd）, 25672（Erlang distribution）
 * 5672，5671（AMQP 0-9-1 without and with TLS）
 * 15672（if management plugin is enabled）
 * 61613，61614（if STOMP is enabled）
 * 1883，8883（if MQTT is enabled）

使用rpm安装完rabbitmq后，默认在/etc/rabbitmq/目录里是没有rabbitmq.config文件的，你可以手动创建，也可以复制一份默认的配置文件(/usr/share/doc/rabbitmq-server-3.5.0/rabbitmq.config.example )

默认只允许guest用户通过localhost本机访问，远程是无法访问的，而一般服务器不安装桌面的，所以我们需要配置允许远程访问.

四。启用管理插件，这样可以通过浏览器访问( [http://www.rabbitmq.com/management.html#configuration](http://www.rabbitmq.com/management.html#configuration))

[shell]rabbitmq-plugins enable rabbitmq_management[/shell]

可以看到15672端口已在监听。

[http://www.rabbitmq.com/access-control.html](http://www.rabbitmq.com/access-control.html)

================================
RabbitMQ

> wget “http://pypi.python.org/packages/source/s/simplejson/simplejson-2.0.9.tar.gz#md5=af5e67a39ca3408563411d357e6d5e47”
> tar zxvf simplejson-2.0.9.tar.gz
> cd simplejson-2.0.9
> python setup.py build
> python setup.py install

—————————————-

[jdk]

> mkdir /usr/local/java/
> cd /usr/local/java/
> wget http://download.oracle.com/otn-pub/java/jdk/6u26-b03/jdk-6u26-linux-i586.bin
> chmod a+x java\_ee\_sdk-5_07-jdk-6u16-linux.bin
> ./java\_ee\_sdk-5_07-jdk-6u16-linux.bin

—————————————-

> [设置/etc/profile，尾部添加]
>
> #jdk
>
> export JAVA_HOME=/opt/SDK/jdk
> export CLASSPATH=.:$JAVA\_HOME/jre/lib/rt.jar:$JAVA\_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
> export PATH=$PATH:$JAVA_HOME/bin
>
> —————————————-
>
> #执行立即生效
>
> source /etc/profile
>
> 或者参考:

—————————————-

\[erlang\]\[/erlang\]

> wget http://www.erlang.org/download/otp\_src\_R14B03.tar.gz
> tar zxvf otp\_src\_R14B03.tar.gz
> cd otp\_src\_R14B03
> ./configure
>
> #忽略警告：wx             : Can not link the wx driver, wx will NOT be useable
>
> make && make install

—————————————-

[rabbitmq]

> wget http://www.rabbitmq.com/releases/rabbitmq-server/v2.5.0/rabbitmq-server-2.5.0.tar.gz
> tar zxvf rabbitmq-server-2.5.0.tar.gz
> cd rabbitmq-server-2.5.0
> make TARGET\_DIR=/usr/local/rabbitmq SBIN\_DIR=/usr/local/rabbitmq/sbin MAN_DIR=/usr/local/rabbitmq/man install

如果在安装rabbitmq的时候,出现”/bin/sh: xsltproc: command not found”错误提示信息,执行 “yum -y install libxslt”
如果报”/bin/sh: line 1: xmlto: command not found”错误,执行”yum -y install xmlto”.

cd /usr/local/rabbitmq/sbin/
rabbitmq-server #rabbitmq-server -detached 后台运行
rabbitmqctl status
rabbitmqctl stop