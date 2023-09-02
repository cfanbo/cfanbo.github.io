---
title: Windows 和 Linux 下生成以当前时间命名的文件
author: admin
type: post
date: 2011-02-28T08:24:57+00:00
url: /archives/7863
IM_contentdowned:
 - 1
categories:
 - 其它
tags:
 - dos

---
在 Windows、Linux 操作系统，分别利用BAT批处理文件和Shell脚本，生成类似“20110228\_082905.txt”以“年月日\_时分秒”命名的文件。

**Windows BAT批处理文件：**

> @echo off
> set time_hh=%time:~0,2%
> if /i %time\_hh% LSS 10 (set time\_hh=0%time:~1,1%)
> set filename=%date:~,4%%date:~5,2%%date:~8,2%\_%time\_hh%%time:~3,2%%time:~6,2%
> echo test >> %filename%.txt

**Linux Shell 脚本：**

> #!/bin/sh
> echo test >> $(date -d “today” +”%Y%m%d_%H%M%S”).txt