---
title: sphinx在windows下无法启动的解决办法
author: admin
type: post
date: 2009-11-17T03:52:04+00:00
url: /archives/2593
IM_data:
 - 'a:1:{s:48:"/wp-content/uploads/2009/11/sphinex-services.jpg";s:48:"/wp-content/uploads/2009/11/sphinex-services.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - coreseek
 - Sphinx

---
配置完成了sphinx,也安装成为系统服务,但在dos提示符下服务的时候错误

> searchd –-install -–config d:/csft3.1/bin/xxxx.conf
>
> 相应的删除服务命令为：
>
> searchd –delete

[![sphinex-services](/wp-content/uploads/2009/11/sphinex-services.jpg)][1]

这里有两种办法:
1.直接把配置文件复制到c:/windows/system32目录里一份就可以了.
2.在安装服务的时候指定配置文件的物理路径(–config d:/csft3.1/bin/csft.conf)

**索引或者查询时提示：ERROR: invalid token in 配置文件 line 1 col 1.**：

该提示表示当前的配置文件的编码不是UTF-8（无BOM头）格式，无法正确解析，请使用编辑软件打开配置文件，另存为UTF-8（无BOM头）格式；

错误的编码格式包括：Unicode、Unicode BOM、Unicode big endian、Unicode 低位在前、UTF-8 + BOM、UTF-8 Signature、UTF-8 包含签名等；

特别注意：Windows自带的记事本(Notepad)或者写字板（WordPad）无法正确保存为所需格式，请勿使用其编辑配置文件；

推荐编辑器： [点击下载Notepad2绿色版](http://www.coreseek.cn/uploads/csft/tool/Notepad2.zip)；使用Notepad2打开配置文件，依次选择：“文件”菜单－－“编码”－－“UTF-8”，然后保存文件（快捷键CTRL+S）即可。

 [1]: /wp-content/uploads/2009/11/sphinex-services.jpg