---
title: Windows下 Apache 性能优化
author: admin
type: post
date: 2010-07-14T09:39:49+00:00
url: /archives/4659
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache

---
一般来说，WinNT系统下使用IIS，而Apache在Linux下应用的比较多，但是依然有很多人在WinNT系统下使用Apache而非IIS，可能是基于对Windows系统的熟悉吧。今天就来说一下在Windows系统下如果优化Apache的性能。

**mpm_winnt.c**是专门针对Windows NT优化的MPM(多路处理模块)，它使用一个单独的父进程产生一个单独的子进程，在这个子进程中轮流产生多个线程来处理请求。也就是说 mpm_winnt只能启动父子两个进程, 不能像Linux下那样同时启动多个进程。

mpm_winnt主要通过ThreadsPerChild和MaxRequestsPerChild两个参数来优化Apache，下面详细来说明一下。

**ThreadsPerChild**

这个参数用于设置每个进程的线程数, 子进程在启动时建立这些线程后就不再建立新的线程了. 一方面因为mpm_winnt不能启动多个进程, 所以这个数值要足够大，以便可以处理可能的请求高峰; 另一方面该参数以服务器的响应速度为准的, 数目太大的反而会变慢。因此需要综合均衡一个合理的数值。
mpm_winnt上的默认值是64, 最大值是1920. 这里建议设置为100-500之间，服务器性能高的话值大一些，反之值小一些。**MaxRequestsPerChild**

该参数表示每个子进程能够处理的最大请求数, 即同时间内子进程数目.设置为零表示不限制, mpm_winnt上的默认值就是0。

官方参考手册中不建议设置为0, 主要基于两点考虑: (1) 可以防止(偶然的)内存泄漏无限进行，从而耗尽内存; (2) 给进程一个有限寿命，从而有助于当服务器负载减轻的时候减少活动进程的数量。

因此这个参数的值更大程度上取决于服务器的内存，如果内存比较大的话可以设置为0或很大的数字，否则设置一个小的数值。需要说明的是，如果这个值设置的太小的话会造成Apache频繁重启，在日志文件中会看到如下的文字：

> Process exiting because it reached MaxRequestsPerChild. Signaling the parent

这样一来降低了Apache的总体性能。

另外，可以通过查看Apache提供的server-status(状态报告)来验证当前所设置数值是否合理，在httpd.conf文件中做如下设置来打开它：

\# 首先需要加载mod_status模块
**LoadModule status\_module modules/mod\_status.so**

\# 然后设置访问的地址

> SetHandler server-status
> Order deny,allow
> Deny from all

\# 如果限制某个IP访问则设置为 Allow from 192.168.1.1
Allow from all

**总结:**
因为Windows NT下Apache只能启动父子两个进程，因此只能通过增大单个进程的线程数以及单个进程能够处理的最大请求数来进行优化。其他优化的参数同Linux系统下是一样的，大家可以加以参考。

下面针对上述两个参数给出一个建议的设置：

>

 ThreadsPerChild 250

 MaxRequestsPerChild 5000