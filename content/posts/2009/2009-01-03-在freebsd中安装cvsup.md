---
title: 在Freebsd中安装CVSup
author: admin
type: post
date: 2009-01-03T06:40:11+00:00
excerpt: |
 在首次运行 CVSup 之前， 务必确认 /usr/ports 是空的！ 如果您之前已经用其他地方安装了一份 Ports 套件，则 CVSup 可能不会自动删除已经在上游服务器上删除掉的补丁文件。

 1.

 安装 net/cvsup-without-gui 软件包：

 # pkg_add -r cvsup-without-gui

 请参见 如何安装 CVSup (第 A.5.2 节) 以了解更多细节。
 2.

 运行 cvsup：

 # cvsup -L 2 -h cvsup.FreeBSD.org /usr/share/examples/cvsup/ports-supfile
 以上参数请见这里
url: /archives/787
IM_contentdowned:
 - 1
categories:
 - 服务器

---
在首次运行 **CVSup** 之前， 务必确认 `/usr/ports` 是空的！ 如果您之前已经用其他地方安装了一份 Ports 套件，则 **CVSup** 可能不会自动删除已经在上游服务器上删除掉的补丁文件。

1. 安装 [`net/cvsup-without-gui`](http://www.freebsd.org/cgi/url.cgi?ports/net/cvsup-without-gui/pkg-descr) 软件包：

   ```
   # pkg_add -r cvsup-without-gui
   ```



    请参见 [如何安装 CVSup](cvsup.html#CVSUP-INSTALL) ( [第 A.5.2 节](cvsup.html#CVSUP-INSTALL)) 以了解更多细节。

2. 运行 `cvsup`：

   ```
   # cvsup -L 2 -h cvsup.FreeBSD.org /usr/share/examples/cvsup/ports-supfile
   以上参数请见这里
   ```



    将 `cvsup.FreeBSD.org` 改为离您较近的 **CVSup** 服务器。 请参见 [CVSup 镜像](cvsup.html#CVSUP-MIRRORS) ( [第 A.5.7 节](cvsup.html#CVSUP-MIRRORS)) 中的镜像站点完整列表。




   > **注意:** 有时可能希望使用自己的 `ports-supfile`， 比如说，不想每次都通过命令行来指定所使用的 **CVSup** 服务器。
   >
   >
   > 1. 这种情况下， 需要以 `root` 身份将 `/usr/share/examples/cvsup/ports-supfile` 复制到新的位置， 例如 `/root` 或您的主目录。
   >
   > 2. 编辑 `ports-supfile`。
   >
   > 3. 把 `CHANGE_THIS.FreeBSD.org` 修改成离您较近的 **CVSup** 服务器。 可以参考 [CVSup 镜像](cvsup.html#CVSUP-MIRRORS) ( [第 A.5.7 节](cvsup.html#CVSUP-MIRRORS)) 中的镜像站点完整列表。
   >
   > 4. 接下来按如下的方式运行 `cvsup`：
   >
   > ```
   > # cvsup -L 2 /root/ports-supfile
   > ```

3. 此后运行 [cvsup(1)](http://www.freebsd.org/cgi/man.cgi?query=cvsup&sektion=1&manpath=FreeBSD+6.2-RELEASE+and+Ports) 命令将下载最近所进行的改动， 并将它们应用到您的 Ports Collection 上，不过这一过程并不重新联编您系统上的 ports。