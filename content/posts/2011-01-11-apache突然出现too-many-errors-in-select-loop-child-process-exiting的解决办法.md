---
title: apache突然出现Too many errors in select loop. Child process exiting的解决办法
author: admin
type: post
date: 2011-01-11T11:29:33+00:00
url: /archives/7480
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - winsock

---

[Fri Mar 13 19:30:08 2009] [notice] Child 2012: Acquired the start mutex.

[Fri Mar 13 19:30:08 2009] [notice] Child 2012: Starting 250 worker threads.

[Fri Mar 13 19:30:08 2009] [notice] Child 2012: Listening on port 80.

[Fri Mar 13 19:30:08 2009] [error] (OS 10038)An operation was attempted on something that is not a socket.  : Too many errors in select loop. Child process exiting.

[Fri Mar 13 19:30:08 2009] [notice] Child 2012: Exit event signaled. Child process is ending.

[Fri Mar 13 19:30:09 2009] [notice] Child 2012: Released the start mutex

[Fri Mar 13 19:30:09 2009] [notice] Child 2012: All worker threads have exited.

[Fri Mar 13 19:30:09 2009] [notice] Child 2012: Child process is exiting

[Fri Mar 13 19:30:09 2009] [notice] Parent: child process exited with status 0 — Restarting.

[Fri Mar 13 19:30:09 2009] [notice] Apache/2.2.11 (Win32) PHP/5.2.5 configured — resuming normal operations

[Fri Mar 13 19:30:09 2009] [notice] Server built: Dec 10 2008 00:10:06

[Fri Mar 13 19:30:09 2009] [notice] Parent: Created child process 748

[Fri Mar 13 19:30:09 2009] [notice] Disabled use of AcceptEx() WinSock2 API

[Fri Mar 13 19:30:10 2009] [notice] Child 748: Child process is running

[Fri Mar 13 19:30:10 2009] [notice] Child 748: Acquired the start mutex.

[Fri Mar 13 19:30:10 2009] [notice] Child 748: Starting 250 worker threads.

[Fri Mar 13 19:30:10 2009] [notice] Child 748: Listening on port 80.

[Fri Mar 13 19:30:10 2009] [error] (OS 10038)An operation was attempted on something that is not a socket.  : Too many errors in select loop. Child process exiting.

[Fri Mar 13 19:30:10 2009] [notice] Child 748: Exit event signaled. Child process is ending.

[Fri Mar 13 19:30:11 2009] [notice] Child 748: Released the start mutex

[Fri Mar 13 19:30:11 2009] [notice] Child 748: All worker threads have exited.

[Fri Mar 13 19:30:11 2009] [notice] Child 748: Child process is exiting

[Fri Mar 13 19:30:11 2009] [notice] Parent: child process exited with status 0 — Restarting.

[Fri Mar 13 19:30:12 2009] [notice] Apache/2.2.11 (Win32) PHP/5.2.5 configured — resuming normal operations

[Fri Mar 13 19:30:12 2009] [notice] Server built: Dec 10 2008 00:10:06

[Fri Mar 13 19:30:12 2009] [notice] Parent: Created child process 4996

[Fri Mar 13 19:30:12 2009] [notice] Disabled use of AcceptEx() WinSock2 API

[Fri Mar 13 19:30:12 2009] [notice] Child 4996: Child process is running

[Fri Mar 13 19:30:12 2009] [crit] (OS 10022)An invalid argument was supplied.  : Child 4996: setup_inherited_listeners(), WSASocket failed to open the inherited socket.

[Fri Mar 13 19:30:12 2009] [crit] Parent: child process exited with status 3 — Aborting.

这样的日志一直在不断的增长，Apache一直在重启。仔细看看，问题出现在这：

(OS 10038)An operation was attempted on something that is not a socket. : Too many errors in select loop. Child process exiting.

是Winsock这出了问题，把Winsock重启恢复下。

>

> netsh winsock reset
>

然后重启下Apache，再打开Web看下，OK了。