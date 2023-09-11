---
title: fckeditor的Error creating folder “” (Can’t create directory)错误的解决办法
author: admin
type: post
date: 2011-11-17T06:14:19+00:00
url: /archives/11978
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - FCKEditor

---
今天使用了fckeditor的编辑器,在下面的环境里测试着没有一点问题的,但上传到服务器上,总是提示

Error creating folder “” (Can’t create directory)

这个错误.经测试发现是 **apache\_lookup\_uri** 函数出的问题，经测试需要获得物理路径，无奈只能修改为 $_SERVER函数

打开文件，fckeditor\editor\filemanager\connectors\php\io.php

修改为

> if ( function\_exists( ‘apache\_lookup_uri’ ) )
> {
> /*zongzong 修改
> $info = apache\_lookup\_uri( $path ) ;
> return $info->filename . $info->path_info ;
> */
> return $\_SERVER[‘DOCUMENT\_ROOT’].$path;
> }

即可.

**apache\_lookup\_uri** 函数参考:[http://docs.haohtml.com/php\_manual\_zh_html/function.apache-lookup-uri.html][1]

 [1]: http://docs.haohtml.com/php_manual_zh_html/function.apache-lookup-uri.html