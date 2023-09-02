---
title: 在FreeBSD下安装cUrl
author: admin
type: post
date: 2010-04-02T01:10:15+00:00
url: /archives/3242
IM_contentdowned:
 - 1
categories:
 - 服务器

---
Before we download the ports collection lets install **curl**, a very useful tool that will help us download the ports archive itself. We do this using the **pkg_add** command.

 `# pkg_add -r curl`

As simple as that. The previous command should download the packages from the remote repo (the -r option stands for “remote”) and install them. If everything goes according to plan you should output that resembles the following:

`Fetching ftp://ftp.freebsd.org/pub/FreeBSD/ports/i386/packages-7.2-release/Latest/curl.tbz... Done.

Fetching ftp://ftp.freebsd.org/pub/FreeBSD/ports/i386/packages-7.2-release/All/ca_root_nss-3.11.9_2.tbz... Done.

`

After installing a package you will need to run **rehash** in order to refresh your environment path if you want to use the command straight away, otherwise it will be available next time you log in.

`# rehash`

Note that the **rehash** command will not show any output.

来源: [http://www.lupomontero.com/managing-ports-in-freebsd/](http://www.lupomontero.com/managing-ports-in-freebsd/)