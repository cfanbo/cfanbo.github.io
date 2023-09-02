---
title: windows下安装 Xdebug＋WinCacheGrind
author: admin
type: post
date: 2010-03-26T07:41:23+00:00
excerpt: |
 以PHP5.1.4，Windows平台为例（其它PHP版本，其它平台请参看官网文档）：
 1． 在http://www.xdebug.org/download.php下载适合自己php版本的php_xdebug- 2.0.1-5.1.2.dll【有附件提供下载，如果按照以下步骤完成后 phpinfo任无法显示xdebug，那么建议重新下载其他xdebug.dll文件试试 】；
 2． 将下载的xdebug.dll放到php\ext目录里,可以重命名也可以不重命名，这里我没有重命名。
 3． 编辑php.ini，加入下面几行：
url: /archives/3096
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - mysql
 - WinCacheGrind
 - xdebug

---
以PHP5.1.4，Windows平台为例（其它PHP版本，其它平台请参看官网文档）：
1． 在 [http://www.xdebug.org/download.php](http://www.xdebug.org/download.php) 下载适合自己php版本的php_xdebug- 2.0.1-5.1.2.dll【有附件提供下载，如果按照以下步骤完成后 phpinfo任无法显示xdebug，那么建议重新下载其他xdebug.dll文件试试 】；
2． 将下载的xdebug.dll放到php\ext目录里,可以重命名也可以不重命名，这里我没有重命名。
3． 编辑php.ini，加入下面几行：

extension=php_xdebug-2.0.1-5.1.2.dll

;xdebug配置
[Xdebug]
;开启自动跟踪
xdebug.auto_trace = On
;开启异常跟踪
xdebug.show\_exception\_trace = On
;开启远程调试自动启动
xdebug.remote_autostart = On
;开启远程调试
xdebug.remote_enable = On
;收集变量
xdebug.collect_vars = On
;收集返回值
xdebug.collect_return = On
;收集参数
xdebug.collect_params = On

xdebug.profiler_enable=on
xdebug.trace\_output\_dir=”D:/php/xdebug”
xdebug.profiler\_output\_dir=”D:/php/xdebug”  //(用来存放性能分析文件,可自由定义目录)

重启Apache；

写一个test.php，内容为，如果输出的内容中有看到xdebug，说明安装配置成功。
 xdebug support

 enabled

 Version

 2.0.2-dev


xdebug+WinCacheGrind【有附件提供下载】将能更好的分析代码,启动WinCacheGrind，然后在 tools—options里设置working folder为刚才xdebug中指定的分析文件目录：D:/php/xdebug即可。

写一个简单的错误代码测试：

>
> testXdebug();
>
> function testXdebug() {
>
> requireFile();
>
> }
> function requireFile() {
>
> require_once(‘abc.php’);
>
> }
> ?>

此时IE页面的报错也很显示，D:/php/xdebug目录下也会出现相应的错误报告文件，用WinCacheGrind打开查看也 很详细。当然xdebug也可以很好的结合eclipse检查报错信息，还有待研究。

- [WinCacheGrind.rar](/wp-content/uploads/2010/12/WinCacheGrind.rar)(445 KB)，最新版本从官方下载： [http://sourceforge.net/projects/wincachegrind/files/](http://sourceforge.net/projects/wincachegrind/files/)

- [php_xdebug-2.0.1-5.1.2.rar](/wp-content/uploads/2010/12/php_xdebug-2.0.1-5.1.2.rar) (54.1 KB),最新版从官方下载： [http://xdebug.org](http://xdebug.org/)