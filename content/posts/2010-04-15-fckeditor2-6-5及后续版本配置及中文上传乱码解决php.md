---
title: FCKeditor2.6.5及后续版本配置及中文上传乱码解决(PHP)
author: admin
type: post
date: 2010-04-15T05:34:57+00:00
url: /archives/3391
IM_contentdowned:
 - 1
categories:
 - 前端设计
tags:
 - FCKEditor

---
[![fckedior_3.1](http://blog.haohtml.com/wp-content/uploads/2010/04/fckedior_3.1.png)][1]
1、首先去官网下载FCKeditor2.6.5 多国语言版。 [http://ckeditor.com/download](http://ckeditor.com/download)，注意： 第一个为最新3.0.1版，第二个才是FCKeditor 2.6.5

2、删除不必要的东西：

删除/FCKeditor/目录下除 fckconfig.js,fckeditor.js,fckstyles.xml,fcktemplates.xml,fckeditor.php,fckeditor\_php5.php,fckeditor\_php4.php
七个文件以外的所有文件；
删除目录/editor/\_source(基本上，所有\_开头的文件夹或文件都是可选的)；
删除/editor/filemanager/connectors/下除了php目录的所有目录；
删除/editor/lang/下的除了 en.js, zh.js, zh-cn.js三个文件的所有文件。

3、打开/FCKeditor/fckconfig.js
修改
var FCKConfig.DefaultLanguage = ‘zh-cn’ ;
var _FileBrowserLanguage   = ‘php’ ;
var _QuickUploadLanguage   = ‘php’ ;
要开启文件上传的话，还需要配置editor\filemanager\connectors\php\config.php
将$Config[‘Enabled’] = false ;改为$Config[‘Enabled’] = true ;
更改$Config[‘UserFilesPath’] = ‘/userfiles/’ ;为你的上传目录；

4.调用方法(例子)
将FCKeditor放在网站根目录
在PHP文件里面，包含/FCKeditor/fckeditor.php文件
//包含fckeditor类
include(“../FCKeditor/fckeditor.php”) ;
//设置编辑器路径
$sBasePath = “/FCKeditor/”;
//创建一个Fckeditor，表单的txtarea名称为content
$oFCKeditor = new FCKeditor(‘content’) ;
$oFCKeditor->BasePath   = $sBasePath ;
//设置表单初始值
$oFCKeditor->Value   = ‘This is some **sample text**’ ;
$oFCKeditor->Create() ;

//还可设置
$oFCKeditor->Width
$oFCKeditor->Height
$oFCKeditor->ToolbarSet
……………………………………………………………………………………………………………………………………
BasePath=’fckeditor/’;
$oFCKeditor->value=’default text in editor’;
$oFCKeditor->Width=’800px’;
$oFCKeditor->Height=’300px’;
$oFCKeditor->create();
//$fck=$oFCKeditor->CreateHtml();
?>
……………………………………………………………………………………………………………………………………

对于Fckeditor上传中文名文件时显示乱码的问题，现公布方法如下：
测试环境：php 5 , utf-8编码

1、修正上传中文文件时文件名乱码问题
在文件connectors/php/commands.php中查找：
$sFileName = $oFile[‘name’] ;
在后面添加一行：
$sFileName = iconv(“utf-8″,”gbk”,$sFileName);

2、修正文件列表时中文文件名显示乱码问题
在文件connectors/php/util.php中查找：
return ( utf8_encode( htmlspecialchars( $value ) ) ) ;
修改为：
return iconv(”,’utf-8′,htmlspecialchars( $value ));

3、修正新建中文文件夹时的文件夹名乱码问题
在文件connectors/php/commands.php中查找：
$sNewFolderName =
在后面添加一行：
$sNewFolderName = iconv(“utf-8″,”gbk”,$sNewFolderName);

2.6.3版及后续版本的fck下的html文件已经加了utf-8的文件头。
[![fckeditor_7](http://blog.haohtml.com/wp-content/uploads/2010/04/fckeditor_7.jpg)][2]

 [1]: /wp-content/uploads/2010/04/fckedior_3.1.png
 [2]: /wp-content/uploads/2010/04/fckeditor_7.jpg