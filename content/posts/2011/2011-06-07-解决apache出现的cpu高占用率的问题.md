---
title: 解决Apache出现的CPU高占用率的问题
author: admin
type: post
date: 2011-06-07T01:17:47+00:00
url: /archives/9687
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - cpu

---
所谓Apache出现CPU高占用率就是指Apache在一段时间内持续占用很高的CPU使用率，甚至达到CPU100％，这个时候造成网站无法访问。解决的方法就是仔细观察Apache的日志文件，查阅错误的信息。

我个人试了一下启用了

> EnableSendfile Off

暂时解决了，

下面我们针对几种错误信息进行分析并给出解决的方法：

**1. Apache与WinSock v2相冲突**
[Apache官方提供的手册](http://httpd.apache.org/docs/2.0/mod/mpm_winnt.html) 中提到，在Windows系统下Apache2.x为了提高性能而使用了Microsoft WinSock v2 API，但是一些常见的防火墙软件会破坏他的正确性，从而使得Apache出现死循环操作造成CPU100％。

其错误提示如下所示：

> \[error\] (730038)An operation was attempted on something that is not a socket.: winnt_accept: AcceptEx failed. Attempting to recover.
>
> \[error\] (OS 10038) : Child 3356: Encountered too many errors accepting client connections. Possible causes: dynamic address renewal, or incompatible VPN or firewall software. Try using the Win32DisableAcceptEx directive.
>
> \[warn\] (OS 121)信号灯超时时间已到。 : winnt_accept: Asynchronous AcceptEx failed.
>
> \[warn\] (OS 64)指定的网络名不再可用。 : winnt_accept: Asynchronous AcceptEx failed.

可以依次采用下面的方法来解决上面的问题，如果进行了一步还有问题就继续下一步：

1) 在httpd.conf文件中使用 Win32DisableAcceptEx 禁止Apache使用 Microsoft WinSock v2 API ：

2. Win32DisableAcceptEx # 禁止使用AcceptEx()


2) 使用 [System Repair Engineer(SREng)](http://www.kztechs.com/sreng/download.html) 查看WinSocket供应者，如果出现非MS的陌生项则将其删除，并使用软件的“重置WinSocket”按钮进行重置。

3) 卸载与Apache相冲突的杀毒软件或防火墙软件。

如果进行上面的三个步骤之后还有问题，那应该看看是不是还有下面的错误。

**2. “Terminating 1 threads that failed to exit”错误**
上面错误中的数字1有可能是其他数字，造成这个错误的原因是Apache在退出并发线程的时候出现线程溢出，从而造成内存泄露，表现出来的就是Apache所占用的系统资源持续增长。

解决的方法也很简单，将最大请求线程的值降低一些，但是也不能太低，太低的话会产生大量的请求队列从而造成站点访问缓慢。如果之前为0则将其设置为一个最大值。

1. MaxRequestsPerChild 10000


**3. “file .\\server\\mpm\\winnt\\child.c, line 1078, assertion “(rv >= 0) && (rv < threads_created)” failed” 错误**

这个错误是Apache的一个 [bug(#11997)](http://issues.apache.org/bugzilla/show_bug.cgi?id=11997)，可以通过 Win32DisableAcceptEx 禁止Apache使用WinSocket v2来避免此bug，具体设置见前述。

**4. PHP5.2.1以上版本的libmysql.dll与MySQL5不兼容**
PHP5.2.1以后的新版本(截止目前最新版本为5.2.5)中用于连接MySQL的libmysql.dll组件与MySQL5不兼容，在Apache中运行PHP的时候会造成Apache产生CPU100%的问题。

解决的方法就是从 [http://www.php.net/releases/](http://www.php.net/releases/) 下载5.2.1版本，将压缩包中的libmysql.dll文件覆盖现在的文件，然后重启Apache就可以了。

**5. 病毒或木马程序命名为Apache.exe**
有的时候病毒或木马程序会将其名称命名为Apache.exe文件达到一种掩饰的目的，这个时候使用第三方进程分析器查看进程的路径然后将其删除或使用杀毒软件清除就可以了。

**6. 程序编写不严谨造成死循环等错误**
如果上面的问题都不存在Apache依然产生CPU100%的问题的话，通常来说就应该是Web程序自身的问题了，例如死循环等等。这个时候需要在日志中设置HTTP请求的文件及执行的时间，然后查找出执行时间比较长的地址进行分析排查。

日志格式设置如下：

> LogFormat “%v %h %l %u %t [%Ts] \”%r\” %>s %b” vhost_common #设置程序执行时间
>
>
> ServerName xxx.xxx.com
> DirectoryIndex index.php index.html index.htm
> DocumentRoot “xxx”
> \# cronolog.exe是Apache自带的用于将日志文件进行分割的应用程序
> CustomLog “|bin/cronolog.exe e:/%Y%m%d.log” vhost_common
>

参考资料：
[apache 遇到[error] (OS 10038) 错误](http://groups.google.com/group/eachgain/browse_thread/thread/203204184f99e544/339ae75ba239e064%23339ae75ba239e064) [Error in my_thread_global_end(): 1 threads didn’t exit](https://support.kayako.com/index.php?_m=knowledgebase&_a=viewarticle&kbarticleid=236) [查看Apache并发请求数及其TCP连接状态](http://blog.s135.com/read.php/269.htm?page=1) [Apache Bug 11997](http://issues.apache.org/bugzilla/show_bug.cgi?id=11997) [关于apache.exe开机占用cpu100%的终极解决方法](http://zhumeng99.cn/read.php?48) [apache cpu占用100%的问题。](http://riddle.bokee.com/6200980.html) [空间持续CPU100%时间过长，如何优化程序](http://x.discuz.net/viewthread-609579.html)