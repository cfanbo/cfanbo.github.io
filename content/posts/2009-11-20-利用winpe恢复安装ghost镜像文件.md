---
title: 利用WINPE恢复安装GHOST镜像文件
author: admin
type: post
date: 2009-11-20T07:35:35+00:00
excerpt: |
 首先，启动WINPE，进入PE系统，至于进入PE系统的方法,可以将下载好的PE映像，刻录成光盘，再用光驱启动运行，也可以用U盘启动运行，具体方法,可见：
 利用老毛桃WinPE制作启动U盘安装系统：
url: /archives/2627
IM_data:
 - 'a:8:{s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_cf4523cf09ba1c9.jpg";s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_cf4523cf09ba1c9.jpg";s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_b6279075cc4851a.jpg";s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_b6279075cc4851a.jpg";s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_c8838e2027fac91.jpg";s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_c8838e2027fac91.jpg";s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_23285a741c63322.jpg";s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_23285a741c63322.jpg";s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_1f4a5cad0552bd0.jpg";s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_1f4a5cad0552bd0.jpg";s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_9af37b3788043cd.jpg";s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_9af37b3788043cd.jpg";s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_4cb1b3989423b8e.jpg";s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_4cb1b3989423b8e.jpg";s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_f046e88254603c4.jpg";s:69:"http://gzweix.com.cn/pcbbs/2009-05-07/1/31_119565_f046e88254603c4.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - winpe

---
首先，启动WINPE，进入PE系统，至于进入PE系统的方法,可以将下载好的PE映像，刻录成光盘，再用光驱启动运行，也可以用U盘启动运行，具体方法,可见： 利用老毛桃WinPE制作启动U盘安装系统：启动到PE后，点开始→程序→克隆工具→诺顿GHOST32 V11，如图

[![](http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_1.jpg)][1]

这样，ghost程序被运行，如图，点OK

[![](http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_2.jpg)][2]

选择local-partition-from image ，如图，也就是从上往下数1－2－3 ，一定注意选择不要搞错了，搞错了会很麻烦的。操作很简单，要细心检查以避免不必要的麻烦。

[![](http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_3.jpg)][3]

用鼠标点击浏览选择盘符和路径

[![](http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_41.jpg)](http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_41.jpg)

找到GHOST镜像文件*.GHO

[![](http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_5.jpg)][4]

点OPEN打开，弹出如下界面，继续OK

[![](http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_6.jpg)][5]

选择要恢复到的磁盘驱动器，在这里，有两个磁盘，一个是U盘，别一个大的是硬盘，选择硬盘

[![](http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_7.jpg)][6]

选择恢复到系统分区C盘

[![](http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_8.jpg)][7]

点击OK，出现提示分区将被重写覆盖。

[![](http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_9.jpg)][8]

点击YES 开始恢复，恢复完成后重启计算机，完成剩余恢复安装，不用你碰一下电脑，便可自动完成安装。

 [1]: http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_1.jpg
 [2]: http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_2.jpg
 [3]: http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_3.jpg
 [4]: http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_5.jpg
 [5]: http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_6.jpg
 [6]: http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_7.jpg
 [7]: http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_8.jpg
 [8]: http://blog.haohtml.com/wp-content/uploads/2009/11/winpe_9.jpg