---
title: centos 5.x 安装 zendOptimizer 3.3.9
author: admin
type: post
date: 2011-04-10T11:04:44+00:00
url: /archives/9180
IM_contentdowned:
 - 1
IM_data:
 - 'a:2:{s:0:"";s:0:"";s:57:"http://s9.sinaimg.cn/middle/4560b80bg91ab73e302e8&690";s:57:"http://blog.haohtml.com/wp-content/uploads/2011/04/adf98.";}'
categories:
 - 服务器
tags:
 - centos
 - selinux

---
刚完成了在CentOS5.5安装Zend Optimizer插件的任务，以前老版本 Zend Optimizer的安装方法是运行安装脚本 ./install.sh，新的Zend Optimizer 3.3.9没有安装脚本，只能按照以下方法安装。

> wget [http://downloads.zend.com/optimizer/3.3.9/ZendOptimizer-3.3.9-linux-glibc23-i386.tar.gz](http://downloads.zend.com/optimizer/3.3.9/ZendOptimizer-3.3.9-linux-glibc23-i386.tar.gz) (32位）
> 或者
> wget [http://downloads.zend.com/optimizer/3.3.9/ZendOptimizer-3.3.9-linux-glibc23-x86_64.tar.gz](http://downloads.zend.com/optimizer/3.3.9/ZendOptimizer-3.3.9-linux-glibc23-x86_64.tar.gz) （64位）

解压缩下载的文件包(x86):

> tar -zxvf ZendOptimizer-3.3.9-linux-glibc23-i386.tar.gz
> cd ZendOptimizer-3.3.9-linux-glibc23-i386
> cd data/5\_2\_x_comp/

这里要注意，进入data文件夹后，so文件是对应版本的，看好PHP版本再安装.把 ZendOptimizer.so 文件拷贝到 /usr/lib/php/modules/ 注意您的目录很可能为/usr/local/php/lib/php/extensions/no-debug-non-zts-20060613/ 或者 /usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/ 请根据实际情况而定!

> **cp ZendOptimizer.so /usr/lib/php/modules/**

把下列两行加入**php.ini**，不要加入任何空格和制表符

> zend\_optimizer.optimization\_level=15
> zend_extension=/usr/lib/php/modules/ZendOptimizer.so

重启Apache/Nginx即可.上面ZendOptimizier.so文件的路径根据实际情况来修改．我这里用的是路径是/usr/local/php/lib/php/extensions/no-debug-non-zts-20060613/ZendOptimizer.so

有可能会遇到＂ZendOptimizer.so: cannot restore segment prot after reloc: Permission denied＂之类的错误．

我用的是Nginx．

执行：

> /usr/local/php/sbin/php-fpm restart

提示：

> Shutting down php\_fpm . doneStarting php\_fpm Failed loading /usr/local/Zend/lib/ZendOptimizer.so: /usr/local/Zend/lib/ZendOptimizer.so: cannot restore segment prot after reloc: Permission denieddone

原来这是SELinux搞的鬼，解决办法有两种:
**法一：**关闭SELINX，执行：

> /usr/sbin/setenforce 0

禁止掉SELinux.

> 更改/etc/sysconfig/selinux 文件内容将 SELINUX=enforcing 改为为 SELINUX=disabled

**法二：**当然不想关闭SWlinux,我们可以这样：

> chcon -t textrel\_shlib\_t /usr/local/zend/ZendOptimizer.so

上面的ZendOptimizer.so根据实际情况修改成绝对的路径即可．

然后再执行 /usr/local/php/sbin/php-fpm restart 即可看到没有了上面的错误提示．

以下面一键安装shell脚本(注意so文件的安装路径,适合32位和64位系统)：

```
#/bin/bash
os=`getconf LONG_BIT`

if [ $os == 32 ]; then
        wget -c http://downloads.zend.com/optimizer/3.3.9/ZendOptimizer-3.3.9-linux-glibc23-i386.tar.gz
        tar -zxvf ZendOptimizer-3.3.9-linux-glibc23-i386.tar.gz
        cp ZendOptimizer-3.3.9-linux-glibc23-i386/data/5_2_x_comp/ZendOptimizer.so /usr/local/php/lib/php/extensions/no-debug-non-zts-20060613/
else
        wget -c http://downloads.zend.com/optimizer/3.3.9/ZendOptimizer-3.3.9-linux-glibc23-x86_64.tar.gz
        tar -zxvf ZendOptimizer-3.3.9-linux-glibc23-x86_64.tar.gz
        cp ZendOptimizer-3.3.9-linux-glibc23-x86_64/data/5_2_x_comp/ZendOptimizer.so /usr/local/php/lib/php/extensions/no-debug-non-zts-20060613/

fi

cat >> /usr/local/php/etc/php.ini <<EOF

[zend]
zend_optimizer.optimization_level=15
zend_extension=/usr/local/php/lib/php/extensions/no-debug-non-zts-20060613/ZendOptimizer.so
EOF
/usr/sbin/setenforce 0
/usr/local/php/sbin/php-fpm restart
```