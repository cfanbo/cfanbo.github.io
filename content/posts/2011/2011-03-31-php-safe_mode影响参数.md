---
title: php Safe_mode影响参数
author: admin
type: post
date: 2011-03-31T15:02:25+00:00
url: /archives/8824
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - php

---

safe_mode是唯一PHP_INI_SYSTEM属性，必须通过php.ini或httpd.conf来设置。要启用safe_mode，只需修改php.ini： safe_mode = On 或者修改httpd.conf，定义目录：

Options FollowSymLinks php_admin_value safe_mode 1重启apache后safe_mode就生效了。启动safe_mode，会对许多PHP函数进行限制，特别是和系统相关的文件打开、命令执行等函数。

所有操作文件的函数将只能操作与脚本UID相同的文件


虽然safe_mode不是万能的（低版本的PHP可以绕过），但还是强烈建议打开安全模式，在一定程度上能够避免一些未知的攻击。不过启用 safe_mode会有很多限制，可能对应用带来影响，所以还需要调整代码和配置才能和谐。被安全模式限制或屏蔽的函数可以参考PHP手册。

 函数名

 限制

 dbmopen()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。

 dbase_open()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。

 filepro()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。

 filepro_rowcount()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。

 filepro_retrieve()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。

 ifx_* sql_safe_mode

 限制(!= safe mode)

 ingres_* sql_safe_mode

 限制(!= safe mode)

 mysql_* sql_safe_mode

 限制(!= safe mode)

 pg_loimport()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。

 posix_mkfifo()

 检查被操作的目录是否与正在执行的脚本有相同的 UID（所有者）。

 putenv()

 遵循 ini 设置的 safe_mode_protected_env_vars 和 safe_mode_allowed_env_vars 选项。请参考 putenv() 函数的有关文档。

 move_uploaded_file()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。

 chdir()

 检查被操作的目录是否与正在执行的脚本有相同的 UID（所有者）。

 dl()

 本函数在安全模式下被禁用。

 backtick operator

 本函数在安全模式下被禁用。

 shell_exec()

 （在功能上和 backticks 函数相同） 本函数在安全模式下被禁用。

 exec()

 只能在 safe_mode_exec_dir 设置的目录下进行执行操作。基于某些原因，目前不能在可执行对象的路径中使用 ..。escapeshellcmd() 将被作用于此函数的参数上。

 system()

 只能在 safe_mode_exec_dir 设置的目录下进行执行操作。基于某些原因，目前不能在可执行对象的路径中使用 ..。escapeshellcmd() 将被作用于此函数的参数上。

 passthru()

 只能在 safe_mode_exec_dir 设置的目录下进行执行操作。基于某些原因，目前不能在可执行对象的路径中使用 ..。escapeshellcmd() 将被作用于此函数的参数上。

 popen()

 只能在 safe_mode_exec_dir 设置的目录下进行执行操作。基于某些原因，目前不能在可执行对象的路径中使用 ..。escapeshellcmd() 将被作用于此函数的参数上。

 fopen()

 检查被操作的目录是否与正在执行的脚本有相同的 UID（所有者）。

 mkdir()

 检查被操作的目录是否与正在执行的脚本有相同的 UID（所有者）。

 rmdir()

 检查被操作的目录是否与正在执行的脚本有相同的 UID（所有者）。

 rename()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。 检查被操作的目录是否与正在执行的脚本有相同的 UID（所有者）。

 unlink()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。 检查被操作的目录是否与正在执行的脚本有相同的 UID（所有者）。

 copy()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。 检查被操作的目录是否与正在执行的脚本有相同的 UID（所有者）。 (on source and target )

 chgrp()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。

 chown()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。

 chmod()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。 另外，不能设置 SUID、SGID 和 sticky bits

 touch()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。 检查被操作的目录是否与正在执行的脚本有相同的 UID（所有者）。

 symlink()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。 检查被操作的目录是否与正在执行的脚本有相同的 UID（所有者）。 （注意：仅测试 target）

 link()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。 检查被操作的目录是否与正在执行的脚本有相同的 UID（所有者）。 （注意：仅测试 target）

 apache_request_headers()

 在安全模式下，以“authorization”（区分大小写）开头的标头将不会被返回。

 header()

 在安全模式下，如果设置了 WWW-Authenticate，当前脚本的 uid 将被添加到该标头的 realm 部分。

 PHP_AUTH 变量

 在安全模式下，变量 PHP_AUTH_USER、PHP_AUTH_PW 和 PHP_AUTH_TYPE 在 $_SERVER 中不可用。但无论如何，您仍然可以使用 REMOTE_USER，来获取用户名称（USER）。（注意：仅 PHP 4.3.0 以后有效）

 highlight_file(), show_source()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。 检查被操作的目录是否与正在执行的脚本有相同的 UID（所有者）。 （注意，仅在 4.2.1 版本后有效）

 parse_ini_file()

 检查被操作的文件或目录是否与正在执行的脚本有相同的 UID（所有者）。 检查被操作的目录是否与正在执行的脚本有相同的 UID（所有者）。 （注意，仅在 4.2.1 版本后有效）

 set_time_limit()

 在安全模式下不起作用。

 max_execution_time

 在安全模式下不起作用。

 mail()

 在安全模式下，第五个参数被屏蔽。（注意，仅自 PHP 4.2.3 起受影响）