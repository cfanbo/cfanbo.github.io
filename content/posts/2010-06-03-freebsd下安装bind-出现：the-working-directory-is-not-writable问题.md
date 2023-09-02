---
title: freebsd下安装BIND 出现：the working directory is not writable问题
author: admin
type: post
date: 2010-06-03T08:57:18+00:00
url: /archives/3798
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - bind

---
**代码:**
 chmod 644 /etc/mtree/BIND.chroot.dist

 vi /etc/mtree/BIND.chroot.dist


 修改：

 /set type=dir uname=root gname=wheel mode=0755

成：
/set type=dir uname=bind gname=wheel mode=0755

转 [http://bbs.chinaunix.net/viewthread.php?tid=1516828](http://bbs.chinaunix.net/viewthread.php?tid=1516828)

我按上面的方法换了一下,好像还需要用chown bind:bind /etc/mtree/BIND.chroot.dist命令来修改文件的属主才可以的．