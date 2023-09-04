---
title: 私密：Linux 内核select、poll 和 eventpoll 的实现
author: admin
type: post
date: 2021-05-01T06:22:35+00:00
draft: true
private: true
url: /?p=30291
categories:
 - 程序开发
tags:
 - Linux

---
Linux 内核仓库 [https://github.com/torvalds/linux](https://github.com/torvalds/linux)

Linux 内核文档： [https://www.kernel.org/doc/html/latest/index.html](https://www.kernel.org/doc/html/latest/index.html)（ [中文](https://www.kernel.org/doc/html/latest/translations/zh_CN/index.html)）

开发工具参考： [https://www.kernel.org/doc/html/latest/dev-tools/index.html](https://www.kernel.org/doc/html/latest/dev-tools/index.html)

也可以使用 VSCode + 插件C/C++ GNU Global

通过前面三个博客可以得知 [**select**](https://blog.csdn.net/weixin_38537730/article/details/104097648)，** [poll](https://blog.csdn.net/weixin_38537730/article/details/104099183)**，** [eventpoll](https://blog.csdn.net/weixin_38537730/article/details/104093556)** 的详细实现，现在来总结对比下它们之间的不同:

 1. **select 流程图**
![93c50bd46eded3432584b819b8c1e7cd](https://blogstatic.haohtml.com/uploads/2021/05/cf1f47a3a2058cac1ffe1376a5825bb8.jpg)
 2. **poll 流程图**
![118c7aba5835cf06e006a1834dbe9f40](https://blogstatic.haohtml.com/uploads/2021/05/05795514077440f88ef0622e88fc6eb1.jpg)
 3. **eventpoll 流程图**
![2a937b08de0c8ed6c32b45ae19399a01](https://blogstatic.haohtml.com/uploads/2021/05/f859094931597fe7298edb417c48c96b.png)
 4. **优缺点总结**
 <1> **监控文件最大数不同**：select和poll都是以数组形式传入药监控的文件句柄，而这个数组是有大小限制的1024个左右(不是很清楚).而epoll则是每add一个文件句柄会new一个新epi出来，挂载在ep的红黑树中，监控的文件个数没有明确限制(可能会受限于系统最大打开文件句柄数)从这点上看，epoll是优于select和poll.
 <2> **copy参数次数不同**：无论是select/poll/epoll都是有对应的系统调用，其参数都会从userspace拷贝到kernelspace,如果监控的文件句柄数很大，select/poll在这方面的耗时会明显多于epoll,因为每次调用select/poll都要重新拷贝一次所有的监控句柄，而epoll则在sys\_epoll\_ctrl的时候添加一次存储在内核数据结构中，后续的sys\_epoll\_wait会通过内核数据找到监控的文件句柄对应的file和监控得到的event.
 <3> **轮询方式的差异**：select 会传入监控句柄的最大句柄，从而监控查询0 ~~ max\_fd之间的所有file的驱动状态来获取想要的文件句柄中想要的event，而poll则只会轮询用户传入的文件句柄集，相比select会少轮询很多file的状态，这点上poll明显优于select.而epoll则是完全异步的方式，哪个有更新会添加到ep->rdlist中,epoll\_wait来取走.当连接数不是很多且每个client都非常活跃的情况下,poll > select > epoll,而当连接数巨大且大多数client都是潜水状态的情况下，epoll > poll > select。

本文摘自： https://blog.csdn.net/weixin_38537730/article/details/104100468