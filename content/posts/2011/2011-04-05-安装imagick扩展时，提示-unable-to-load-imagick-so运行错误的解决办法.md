---
title: 安装Imagick扩展时，提示 unable to load imagick.so运行错误的解决办法
author: admin
type: post
date: 2011-04-05T17:27:02+00:00
url: /archives/9010
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ImageMagick

---
> wget [ftp://mirror.aarnet.edu.au/pub/imagemagick/ImageMagick-6.5.5-6.tar.gz][1]

 tar zxvf ImageMagick-6.5.5-6.tar.gz

 cd ImageMagick-6.5.5-6

 ./configure

 make

 make install

 cd ..按照以上方法安装ImageMagick后，有可能会遇到PHP加载imagick.so后运行错误，解决方法是在编译ImageMagick时关掉openmp: –-disable-openmp。如果还不行的话，请更换ImageMagick至低版本，比如：6.5.4-2。



 [1]: ftp://mirror.aarnet.edu.au/pub/imagemagick/ImageMagick-6.5.5-6.tar.gz