---
title: mac下安装GD库FreeType
author: admin
type: post
date: 2015-05-29T12:22:37+00:00
url: /archives/15687
categories:
 - 其它

---
MacBook Pro安装的新系统10.10.3，PHP环境也是默认就有的，GD库在默认情况下也安装过了，但在使用验证码的时候，提示GD库不支持FreeType，这里我们手动安装一下。

**法一：**

## 安装 FreeType

前往苹果官方开源支持：http://www.apple.com/opensource/ 查找并下载GD需要的 zlib/libpng/jpeg/freetype/libgd，这里提供一个包及执行脚本：

[百度网盘下载](http://pan.baidu.com/s/1ntDQ5Tn) 密码:3euq

也可以单个下载安装，例如：

[shell]curl -O http://download.savannah.gnu.org/releases/freetype/freetype-2.4.4.tar.bz2
tar -zxf gd.tar.gz
cd gd
sudo ./install
[/shell]

然后刷新一下 phpinfo(); 或者看一下php支持的库

```
php -m

```

仍然没有看到 FreeType的踪影，因为这些库仅仅是安装了，但仍需要重新编译PHP，才能启用。

接下来就是重新编译PHP，添加 FreeType 支持，因为原PHP中已经编译GD，重新编译GD一定要加入–with-freetype，否则在PHP上仍然不能获得Freetype支持。

提醒：原始的PHP编译使用的参数可以在 phpinfo.php 信息里查看，也可以通过命令

[shell]php -i | grep configure[/shell]

查看，记得在最后添加上–with-freetype 参数，重新编译即可。

**法二：**

**注意：** 使用此方法基于上等于重新安装了PHP版本，只是编译参数包含了以前的参数，我这里升级后，PHP版本升级到了5.5.25.新的安装目录改为了/usr/local/php5。在终端里使用的PHP(5.5.20)和网页使用的PHP版本(5.5.25)为两个不同的版本。 [http://php-osx.liip.ch/](http://php-osx.liip.ch/)
这个网址，里面有任意版本的php安装，而且不影响原来的，今天试了下，不用重新的编译php,
我装的php5.5.20 就这么一句就搞定了 curl -s [http://php-osx.liip.ch/install.sh](http://php-osx.liip.ch/install.sh) | bash -s 5.5