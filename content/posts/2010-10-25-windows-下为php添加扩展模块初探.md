---
title: Windows 下为PHP添加扩展模块初探
author: admin
type: post
date: 2010-10-25T05:44:08+00:00
url: /archives/6338
IM_data:
 - 'a:2:{s:100:"http://blog.zol.com.cn/showpic.php?img=http://bbs.nettf.net/forums/uploads/post-12125-1168699498.gif";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/03/a11c_post-12125-1168699498.gif";s:100:"http://blog.zol.com.cn/showpic.php?img=http://bbs.nettf.net/forums/uploads/post-12125-1168699514.gif";s:81:"http://blog.haohtml.com/wp-content/uploads/2011/03/e3ae_post-12125-1168699514.gif";}'
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php扩展

---
说明：

本人新手，虽然用过很长时间的php，也写过一些简单php程序，但是一直没有对php的扩展模块有过研究。最近因为开发需要，要给一些php应用扩展功能，虽然手边有以前开发好的C程序，用popen等运行之也可以使用，但是从效率、调用便捷性、代码完整性等方面考虑总是觉得popen方式有些不妥，因此萌生了写个扩展模块的念头。于是乎上网找资料，并且初步完成了一个最基本的php扩展模块的框架。在此特别感谢花总的友情支持。

btw: 本文所说的相关技术已经很陈旧了，实在不适合用“初探”这个词，但是于我个人而言，却又的确是初探，现总结出来，分享之。

**0、环境说明**

框架生成环境：

FreeBSD 6.0-STABLE ( 用各版本 Linux 或者 cygwin 也可以,不过我手边只有 bsd,就用它了 )

PHP-4.4.4 源码包 (php-4.4.4.tar.bz2 或 php-4.4.4.tar.gz)

开发工具:

VC++ 6.0 ( 我没有用 VS .Net,因为 VC++6.0 启动比较快些,而且只是写个DLL而已 )

运行环境：

Windows 2003

PHP-4.4.4

Apache 1.3.37

**1、框架生成:**

我在FreeBSD 上下载了 PHP-4.4.4的源码，并将其解包

代码

[neil@Gatwway] ~/phpext >tar -jxvf php-4.4.4.tar.bz2


[neil@Gatwway] ~/phpext >ls -al


total 4406


drwxr-xr-x     3 neil    wheel        512 Jan 10 12:40 .


drwxr-xr-x    13 neil    wheel       1024 Jan 10 12:37 ..


drwxr-xr-x    18 neil    wheel       2048 Jan 10 16:01 php-4.4.4


 -rw-r–r–     1 neil    wheel    4478698 Aug 15 20:32 php-4.4.4.tar.bz2


[neil@Gatwway] ~/phpext >


我们需要php源码包提供的ext_skel工具来生成框架,进入 php-4.4.4/ext 目录

代码

[neil@Gatwway] ~/phpext >cd php-4.4.4/ext/


[neil@Gatwway] ~/phpext/php-4.4.4/ext >ls -al ext_skel


 -rwxr-xr-x    1 neil    wheel    7802 Oct 18    2003 ext_skel


[neil@Gatwway] ~/phpext/php-4.4.4/ext >


运行 ext\_skel 生成框架 ( 假设我们需要制作的扩展模块名字为 nsms\_ext )

代码

[neil@Gatwway] ~/phpext/php-4.4.4/ext >./ext_skel –extname=nsms_ext


Creating directory nsms_ext


Creating basic files: config.m4 .cvsignore nsms_ext.c php_nsms_ext.h CREDITS EXPERIMENTAL tests/001.phpt nsms_ext.php [done].


…


…


[neil@Gatwway] ~/phpext/php-4.4.4/ext >


至此我们需要的框架已经生成完毕了( nsms\_ext/nsms\_ext.c 和 nsms\_ext/php\_nsms_ext.h )

代码

[neil@Gatwway] ~/phpext/php-4.4.4/ext >ls -al nsms_ext/


total 26


drwxr-xr-x     3 neil    wheel     512 Jan 12 09:57 .


drwxr-xr-x    90 neil    wheel    1536 Jan 12 09:58 ..


…


 -rw-r–r–     1 neil    wheel    5380 Jan 12 09:57 nsms_ext.c


…


 -rw-r–r–     1 neil    wheel    2726 Jan 12 09:57 php_nsms_ext.h


…


[neil@Gatwway] ~/phpext/php-4.4.4/ext >


**2、创建Windows DLL工程**

在Windows平台启动 VC++ 6.0, File->New, 在 Project中选择 “Win32 Dynamic-Link Library”, 填写上 Project Name 和 Location, 在第二屏选中 “An empty DLL project.”, 点 Finish 完成创建。

( 我这里创建的 Project Name 为 nsms_ext )

将 “1.框架生成:” 步骤中生成的 nsms\_ext/nsms\_ext.c 和 nsms\_ext/php\_nsms\_ext.h 复制到Windows平台，添加进新创建的工程 nsms\_ext

检查PHP开发环境,如果没有以下目录(.zip或者installer发布的Windows 版本php 貌似不带开发环境)

ext、main、regex、TSRM、win32、Zend

则将 bsd 下的源码的相同目录复制到 Windows 上PHP的根目录下

（假设Windows平台上的PHP安装在 D:PHP4 ）

(*)修改工程设置：

a). Project->Settings

选择 “C/C++” Tab项

在 “Project Options:” 最后加上 /Tc

在 “Preprocessor definitions:” 中添加以下宏定义

ZEND\_DEBUG=0,COMPILE\_DL\_NSMS\_EXT,ZTS=1,ZEND\_WIN32,PHP\_WIN32,HAVE\_NSMS\_EXT=1

**特别注意** COMPILE\_DL\_NSMS\_EXT 和 HAVE\_NSMS\_EXT=1，后面的 “NSMS\_EXT” 要和你创建的 Project Name 一致，我这里是 nsms_ext

如果这里不对 运行 php -v 进行检测时将会出现如下错误：

代码

D:php4>php -v


PHP Warning:    Unknown(): Invalid library (maybe not a PHP library) ‘nsms_ext.dll’    in Unknown on line 0


PHP 4.4.4 (cgi-fcgi) (built: Aug 16 2006 01:17:43)


Copyright (c) 1997-2006 The PHP Group


Zend Engine v1.3.0, Copyright (c) 1998-2004 Zend Technologies


选择 “Link” Tab项

在 “Object/library modules:” 中添加 php4ts.lib

b). Tools->Options

选择 “Directories”

在 “Show directories for:” 下拉框中选择 “Library files”

在 “Directories” 中添加 D:PHP4 （即 php4ts.lib 所在目录）

在 “Show directories for:” 下拉框中选择 “Include files”

在 “Directories” 中添加 D:PHP4 （即 ext、regex、win32 所在目录）

在 “Directories” 中添加 D:PHP4MAIN

在 “Directories” 中添加 D:PHP4TSRM

在 “Directories” 中添加 D:PHP4ZEND

c). Build->Set Active Configuration…

选择 “nims_ext – Win32 Release”

**3、代码框架简单说明**

nsms_ext.c

我们可以找到 “zend\_function\_entry” 部分，这里是定义扩展模块入口函数的结构数组，我这里的代码如下

代码

zend_function_entry nsms_ext_functions[] = {

PHP_FE(nsms_ext_init, NULL)


PHP_FE(nsms_ext_getinfo, NULL)


//          PHP_FE(confirm_nsms_ext_compiled, NULL)    /* For testing, remove later. */


{NULL, NULL, NULL} /* Must be the last line in nsms_ext_functions[] */


};


php\_nsms\_ext.h

我们可以找到用 PHP_FUNCTION(…);进行函数声明的部分，我这里的代码如下

代码

//PHP_FUNCTION(confirm_nsms_ext_compiled); /* For testing, remove later. */


// Declare nsms functions


PHP_FUNCTION(nsms_ext_init);


PHP_FUNCTION(nsms_ext_getinfo);


入口和结构都添加修改好了之后，我们就可以在 nsms_ext.c 中（或者在我们自己的C文件中，看个人的工程的组织结构了）添加声明的函数的实现部分，我这里如下（我是放在其他的C文件中去实现的）

代码

/* {{{ proto string nsms_ext_init( void )

Return a string to get the module infos. */


PHP_FUNCTION(nsms_ext_getinfo)


{


int len;


char string[256];


len = sprintf(string, “License to:%snCient API version:%snProvider:%snAuthor:%s”, NSMS_EXT_G(lpLicenseTo), NSMS_EXT_G(lpAPIver), NSMS_EXT_G(lpProvider), NSMS_EXT_G(lpAuthor) );


RETURN_STRINGL(string, len, 1);


}


/* }}} */


/* {{{ proto string nsms_ext_init(string arg)


Return a string to test that the module is init done. */


PHP_FUNCTION(nsms_ext_init)


{


char *arg = NULL;


int arg_len, len;


char string[256];


if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, “s”, &arg, &arg_len) == FAILURE) {


return;


}


len = sprintf(string, “function nsms_ext_init is called with arg “%s”.”, arg);


RETURN_STRINGL(string, len, 1);


}


/* }}} */


注：lpLicenseTo, lpAPIver, lpProvider, lpAuthor 是我用到的字符串变量，个人按自己的实际情况替换或修改。

至此，一个简单的扩展模块的框架就完成了，编译后将生成的 dll 文件复制到 php 的 extensions 目录下，并在 php.ini 中加载它。运行 php -v 进行测试，如果没有报错或者警告，就可以写一个简单的php测试脚本来看看结果了。

代码

if( ! extension_loaded( ‘nsms_ext’ ) ) {

echo “Module nsms_ext is not loaded.”;


} else {


$ret = nsms_ext_init( “测试一下” ) . “

” . nsms_ext_getinfo();


echo $ret;


}


?>


我这里的输出如下

代码

function nsms_ext_init is called with arg “测试一下”.


License to:[被屏蔽 ^_*]


Cient API version:3.2.0.2


Provider:[被屏蔽 *_^]


Author:Neil


**4、关于 php.ini**

有时候我们的程序想通过配置文件进行一些初始化信息的读取及设置。作为扩展模块，如果每个模块都维护自己的一份配置文件，那简直太混乱了，因此我们使用 php.ini 作为大家共用的配置文件。

对于 nsms_ext 模块，我在 php.ini 中添加了如下项

代码

[NSMS_EXT]


; NSMS_EXT Module Port


nsms_ext.com_port = COM1


下面我们来修改源文件以配合 php.ini 中的扩展

php\_nsms\_ext.h

我们找到 ZEND\_BEGIN\_MODULE\_GLOBALS(nsms\_ext)，这里是定义与配置文件php.ini中扩展的属性匹配的变量的地方，注意名称要一致，另外，一共有2种类型：字符型和长正型，我们需要把原有的注释去掉，如下（我没用到长整型，因此注释掉了）

代码

ZEND_BEGIN_MODULE_GLOBALS(nsms_ext)

//long    global_value;


char * com_port;


ZEND_END_MODULE_GLOBALS(nsms_ext)


nsms_ext.c

取消 ZEND\_DECLARE\_MODULE\_GLOBALS(nsms\_ext) 的注释

代码

/* If you declare any globals in php_nsms_ext.h uncomment this:*/


ZEND_DECLARE_MODULE_GLOBALS(nsms_ext)


/**/


注意：如果你把用PHP\_FUNCTION(…)声明的函数放在了其他文件中，而在这个函数中你又想使用 NSMS\_EXT\_G(v) 这个宏来访问全局变量，那么你需要把 ZEND\_DECLARE\_MODULE\_GLOBALS(nsms_ext) 宏放到一个类似于 global.h 的头文件中，让需要访问全局变量的文件都能引用这个宏定义。

取消 PHP\_INI\_BEGIN 和 PHP\_INI\_END 的注释

代码

/* {{{ PHP_INI


*/


/* Remove comments and fill if you need to have entries in php.ini*/


PHP_INI_BEGIN()

//STD_PHP_INI_ENTRY( “nsms_ext.global_value”,        “42”, PHP_INI_ALL, OnUpdateInt, global_value, zend_nsms_ext_globals, nsms_ext_globals)


STD_PHP_INI_ENTRY( “nsms_ext.com_port”, “COM1”, PHP_INI_ALL, OnUpdateString, com_port, zend_nsms_ext_globals, nsms_ext_globals)


PHP_INI_END()


/**/


/* }}} */


取消 static void php\_nsms\_ext\_init\_globals(zend\_nsms\_ext\_globals *nsms\_ext_globals) 的注释

代码

/* {{{ php_nsms_ext_init_globals


*/


/* Uncomment this function if you have INI entries*/


static void php_nsms_ext_init_globals(zend_nsms_ext_globals *nsms_ext_globals)


{

//nsms_ext_globals- >global_value = 0;


nsms_ext_globals- >com_port = NULL;


}


/**/


/* }}} */


取消 PHP\_MINIT\_FUNCTION(nsms\_ext) 中 ZEND\_INIT\_MODULE\_GLOBALS 和 REGISTER\_INI\_ENTRIES 的注释

代码

/* {{{ PHP_MINIT_FUNCTION


*/


PHP_MINIT_FUNCTION(nsms_ext)


{

/* If you have INI entries, uncomment these lines */


ZEND_INIT_MODULE_GLOBALS(nsms_ext, php_nsms_ext_init_globals, NULL);


REGISTER_INI_ENTRIES();


/**/


return SUCCESS;


}


/* }}} */


取消 PHP\_MSHUTDOWN\_FUNCTION(nsms\_ext) 中 UNREGISTER\_INI_ENTRIES 的注释

代码

/* {{{ PHP_MSHUTDOWN_FUNCTION


*/


PHP_MSHUTDOWN_FUNCTION(nsms_ext)


{

/* uncomment this line if you have INI entries*/


UNREGISTER_INI_ENTRIES();


/**/


return SUCCESS;


}


取消 PHP\_MINFO\_FUNCTION(nsms\_ext) 中 DISPLAY\_INI_ENTRIES 的注释

代码

/* {{{ PHP_MINFO_FUNCTION


*/


PHP_MINFO_FUNCTION(nsms_ext)


{

php_info_print_table_start();


php_info_print_table_header( 2, “nsms_ext support”, “enabled” );


php_info_print_table_row( 2, “License to”,     NSMS_EXT_G(lpLicenseTo) );


php_info_print_table_row( 2, “Client API version”, NSMS_EXT_G(lpAPIver) );


php_info_print_table_row( 2, “Provider”,     NSMS_EXT_G(lpProvider) );


php_info_print_table_row( 2, “Author”,      NSMS_EXT_G(lpAuthor) );


php_info_print_table_end();


/* Remove comments if you have entries in php.ini*/


DISPLAY_INI_ENTRIES();


/**/


}


/* }}} */


至此，我们的基本框架修改已经全部完成了，后面的扩展就看个人的项目需求以及其他需要了。

将修改好的代码编译，关闭apache，将dll复制到 php 的 extensions 目录，启动apache，我们再写个简单的php测试程序来看效果

代码

![](http://blog.zol.com.cn/showpic.php?img=http://bbs.nettf.net/forums/uploads/post-12125-1168699498.gif)

把 php.ini 中的

代码

nsms_ext.com_port = COM1


修改为

代码

nsms_ext.com_port = COM-ABCDEFG


![](http://blog.zol.com.cn/showpic.php?img=http://bbs.nettf.net/forums/uploads/post-12125-1168699514.gif)

**



注：在扩展模块中的内存分配、释放等操作要使用PHP Zend API提供的接口，千万不要直接使用操作系统提供的函数接口，否则会引起扩展模块崩溃甚至php进程崩溃。具体说明可参看下面列出的参考文章

[参考]

1、http://book.itzero.com/read/others/0503/Prentice.Hall.PTR.PHP.5.Power.Programming.Oct.2004.eBook-LiB_html/013147149X/ch15lev1sec2.html

btw: 代码中的 t 全部被过滤掉了。。。只好用空格来修改排版。。。