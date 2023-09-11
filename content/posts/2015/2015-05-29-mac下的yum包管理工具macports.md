---
title: mac下的yum包管理工具MacPorts
author: admin
type: post
date: 2015-05-29T12:31:56+00:00
url: /archives/15691
categories:
 - 其它
tags:
 - homebrew
 - macports

---
这里推荐安装 [Homebrew](http://brew.sh/)，好像安装这个的用户比较的多的，安装命令也非常的简单。

Mac下面除了用dmg、pkg来安装软件外，比较方便的还有用MacPorts来帮助你安装其他应用程序，跟BSD中的ports道理一样。MacPorts就像apt-get、yum一样，可以快速安装些软件。

> 除了这个还有一些类似的工具： [Homebrew](http://brew.sh/) 和 Fink。
>
> Flink是直接编译好的二进制包，MacPorts是下载所有依赖库的源代码，本地编译安装所有依赖，Homebrew是尽量查找本地依赖库，然后下载包源代码编译安装。
> Flink容易出现依赖库问题，MacPorts相当于自己独立构建一套，下载和编译的东西太多太麻烦，Homebrew的方式最合理。

下面将MacPorts的安装和使用方法记录在这里以备查。

访问官方网站http://www.macports.org/install.php，这里提供有dmg安装和源码安装两种方式，dmg就多说了，下载 [MacPorts-2.3.3-10.10-Yosemite.pkg](https://distfiles.macports.org/MacPorts/MacPorts-2.3.3-10.10-Yosemite.pkg)，下一步下一步安装即可。

**通过Source安装MacPorts**

wget http://distfiles.macports.org/MacPorts/MacPorts-1.9.2.tar.gz

tar zxvf MacPorts-1.9.2.tar.gz

cd MacPorts-1.9.2

./configure && make && sudo make install

cd ../

rm -rf MacPorts-1.9.2*

然后将/opt/local/bin和/opt/local/sbin添加到$PATH搜索路径中


编辑/etc/profile文件中，加上

export PATH=/opt/local/bin:$PATH

export PATH=/opt/local/sbin:$PATH

**MacPorts使用**
更新ports tree和MacPorts版本，强烈推荐第一次运行的时候使用-v参数，显示详细的更新过程。
sudo port -v selfupdate

搜索索引中的软件
port search name

安装新软件
sudo port install name

卸载软件
sudo port uninstall name

查看有更新的软件以及版本
port outdated

升级可以更新的软件
sudo port upgrade outdated

Eclipse的插件需要subclipse需要JavaHL，下面通过MacPorts来安装
sudo port install subversion-javahlbindings