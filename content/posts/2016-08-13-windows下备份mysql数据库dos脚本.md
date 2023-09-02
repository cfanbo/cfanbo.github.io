---
title: windows下备份mysql数据库dos脚本
author: admin
type: post
date: 2016-08-13T04:47:58+00:00
url: /archives/17085
categories:
 - 程序开发
tags:
 - dos

---

```
@echo off & setlocal ENABLEEXTENSIONS
:: ---------- 配置项 ----------
:: 备份放置的路径，加
set BACKUP_PATH=D:\Backup
:: 要备份的数据库名称，多个用空格分隔
set DATABASES=1688 abc ceshi
:: MySQL 用户名
set USERNAME=root
:: MySQL 密码
set PASSWORD=root
:: MySQL Bin 目录，加
:: 如果可以直接使用 dump（安装时添加 MySQL Bin 目录到了环境变量），此处留空即可
set MYSQL=D:\Progra~1\phpStudy\MySQL\bin\
:: WinRAR 自带命令行工具的可执行文件路径，长文件名注意用 Dos 长文件名书写方式
set WINRAR=D:\Progra~1\WinRAR\Rar.exe
:: ---------- 以下请勿修改 ----------
set YEAR=%date:~0,4%
set MONTH=%date:~5,2%
set DAY=%date:~8,2%
:: 如果在 dos 下输入 time 返回的不是 24 小时制（没有 0 填充），请自行修改此处
set HOUR=%time:~0,2%
set MINUTE=%time:~3,2%
set SECOND=%time:~6,2%
set DIR=%BACKUP_PATH%\%YEAR%-%MONTH%-%DAY%_%HOUR%%MINUTE%%SECOND%

set ADDON=%YEAR%%MONTH%%DAY%%HOUR%%MINUTE%%SECOND%
:: create dir
if not exist %DIR% (
mkdir %DIR% 2>nul
)

if not exist %DIR% (
echo Backup path: %DIR% not exists, create dir failed.
goto exit
)
cd /d %DIR%

:: backup
echo Start dump databases...
for %%D in (%DATABASES%) do (
echo Dumping database %%D ...
%MYSQL%mysqldump -u%USERNAME% -p%PASSWORD% %%D > %%D.%ADDON%.sql
:: winrar
if exist %WINRAR% (
%WINRAR% a -k -r -s -m1 -ep1 %%D.%ADDON%.rar %%D.%ADDON%.sql 2>null
del /F /S /Q %%D.%ADDON%.sql 2>null
)
)
echo Done
:exit

```