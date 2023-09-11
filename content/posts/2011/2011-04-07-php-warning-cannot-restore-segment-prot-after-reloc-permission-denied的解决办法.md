---
title: 'PHP Warning: cannot restore segment prot after reloc: Permission denied的解决办法'
author: admin
type: post
date: 2011-04-07T14:22:17+00:00
url: /archives/9073
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux
 - selinux

---

Failed loading /usr/local/Zend/lib/Optimizer-3.3.3/php-5.2.x/ZendOptimizer.so: /usr/local/Zend/lib/Optimizer-3.3.3/php-5.2.x/ZendOptimizer.so: cannot restore segment prot after reloc: Permission denied


原来这是SELinux搞的鬼，解决办法有如下两个

**1. 使用chcon 命令**

示例: chcon -t texrel_shlib_t    /usr/local/Zend/lib/Optimizer-3.3.3/php-5.2.x/ZendOptimizer.so

**2. 禁止掉SELinux**

更改/etc/sysconfig/selinux 文件的内容为 SELINUX=disabled


这个GD库的问题,在装好后启动apache的时候,还会提示php库的问题,用上面的同样方法处理即可.