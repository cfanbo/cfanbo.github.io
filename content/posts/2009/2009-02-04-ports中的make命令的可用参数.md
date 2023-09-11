---
title: ports中的make命令的可用参数
author: admin
type: post
date: 2009-02-04T05:51:31+00:00
excerpt: |
 我们经常使用ports来安装程序，ports中的make命令还可以有很多的功能：

 引用
 fetch - Retrieves ${DISTFILES} (and ${PATCHFILES} if defined) into ${DISTDIR} as necessary.
 fetch-list - Show list of files that would be retrieved by fetch.
 fetch-recursive - Retrieves ${DISTFILES} (and ${PATCHFILES} if defined), for port and dependencies into ${DISTDIR} as necessary.
url: /archives/988
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - ports

---
我们经常使用ports来安装程序，ports中的make命令还可以有很多的功能：

引用


**fetch** – Retrieves ${DISTFILES} (and ${PATCHFILES} if defined) into ${DISTDIR} as necessary.

**fetch-list** – Show list of files that would be retrieved by fetch.

**fetch-recursive** – Retrieves ${DISTFILES} (and ${PATCHFILES} if defined), for port and dependencies into ${DISTDIR} as necessary.

**fetch-recursive-list** – Show list of files that would be retrieved by fetch-recursive.


**fetch-required**– Retrieves ${DISTFILES} (and ${PATCHFILES} if defined), for port and dependencies that are not already installed into ${DISTDIR}.

**all-depends-list** – Show all directories which are dependencies for this port.

**build-depends-list**– Show all directories which are build-dependencies for this port.

**package-depends-list** – Show all directories which are package-dependencies for this port.

**run-depends-list** – Show all directories which are run-dependencies for this port.

**extract** – Unpacks ${DISTFILES} into ${WRKDIR}.

**patch** – Apply any provided patches to the source.

**configure** – Runs either GNU configure, one or more local configure scripts or nothing, depending on what’s available.

**build**– Actually compile the sources.

**install** – 安装编译结果.

**reinstall** – 安装编译结果，忽略“已经安装”错误.

**deinstall**– 卸载这个安装.

**deinstall-all** – Remove all installations with the same PKGORIGIN.

**package** – Create a package from an _installed_ port.

**package-recursive** – Create a package for a port and _all_ of its dependancies.

**describe**– Try to generate a one-line description for each port for use in INDEX files and the like.

**checkpatch** – Do a “patch -C” instead of a “patch”. Note that it may give incorrect results if multiple patches deal with the same file.

**checksum** – Use distinfo to ensure that your distfiles are valid.

**checksum-recursive** – Run checksum in this port and all dependencies.

**makesum** – Generate distinfo (only do this for your own ports!).

**clean** – Remove ${WRKDIR} and other temporary files used for building.

**clean-depends** – Do a “make clean” for all dependencies.

**config** – Configure options for this port (using ${DIALOG}). Automatically run prior to extract, patch, configure, build, install, and package.

**showconfig** – 显示这个port的config选项

**rmconfig** – 从这个port移除config选项