---
title: Linux性能测试工具Lmbench介绍和使用说明
author: admin
type: post
date: 2011-10-18T05:56:42+00:00
url: /archives/11740
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux
 - Lmbench

---
Linux性能测试工具Lmbench是一套简易可移植的，符合ANSI/C标准为UNIX/POSIX而制定的微型测评工具。一般来说，它衡量两个关键特征：反应时间和带宽。Lmbench旨在使系统开发者深入了解关键操作的基础成本。

**1、Lmbench的使用与介绍**

Linux性能测试工具Lmbench是一套简易可移植的，符合ANSI/C标准为UNIX/POSIX而制定的微型测评工具。一般来说，它衡量两个关键特征：反应时间和带宽。Lmbench旨在使系统开发者深入了解关键操作的基础成本。其官方网站是：http://www.bitmover.com/lmbench/。
**2、Lmbench主要功能**
带宽测评工具反应时间测评工具其他读取缓存文件
拷贝内存
读内存
写内存
管道
TCP上下文切换
网络：连接的建立，管道，TCP，UDP和RPChotpotato
文件系统的建立和删除
进程创建
信号处理
上层的系统调用
内存读入反应时间处理器时钟比率计算


**3、Linux性能测试工具Lmbench主要特性**
a)对于操作系统的可移植性测试：评测工具是由C语言编写的，具有较好的可移植性（尽管它们更易于被GCC编译）。这对于产生系统间逐一明细的对比结果是有用的。
b)自适应调整：Linux性能测试工具Lmbench对于应激性行为是非常有用的。当遇到BloatOS比所有竞争者慢4倍的情况时，这个工具会将资源进行分配来修正这个问题。
c)数据库计算结果：数据库的计算结果包括了从大多数主流的计算机工作站制造商上的运行结果。
d)存储器延迟计算结果：存储器延迟测试展示了所有系统（数据）的缓存延迟，例如一级，二级和三级缓存，还有内存和TLB表的未命中延迟。另外，缓存的大小可以被正确划分成一些结果集并被读出。硬件族与上面的描述相象。这种测评工具已经找到了操作系统分页策略的中的一些错误。
e)上下文转换计算结果：很多人好象喜欢上下文转换的数量。这种测评工具并不是特别注重仅仅引用“在缓存中”的数量。它时常在进程数量和大小间进行变化，并且在当前内容不在缓存中的时候，将结果以一种对用户可见的方式进行划分。您也可以得到冷缓存上下文切换的实际开销。
f)回归测试：
(一)Sun公司和SGI公司已经使用这种测评工具以寻找和补救存在于性能上的问题。
(二)Intel公司在开发P6的过程中，使用了它们。
(三)Linux在Linux的性能调整中使用了它们。
g)新的测评工具：源代码是比较小的，可读并且容易扩展。它可以按常规组合成不同的形式以测试其他内容。举例来说，如包括处理连接建立的库函数的网络测量，服务器关闭等。
**4、安装与使用**
安装使用Linux性能测试工具Lmbench的安装相对比较简单，到其官方网站下载压缩包Lmbench.tar.gz将其解压，并进入解压后的目录命令行键入makeresults即可开始编译测试。这里需要注意如果在make的时候出错，提示类似

 1. $makeresults
 2. make[1]:Enteringdirectory\`/home/kyuan/lmbench3/src’
 3. gmake[2]:Enteringdirectory\`/home/kyuan/lmbench3/src’
 4. gmake[2]:\***Noruletomaketarget\`../SCCS/s.ChangeSet’,neededbybk.ver’..
 5. gmake[2]:Leavingdirectory\`/home/kyuan/lmbench3/src’
 6. make[1]:\***[lmbench]Error2
 7. make[1]:Leavingdirectory\`/home/kyuan/lmbench3/src’
 8. make:\***[results]Error2

这是需要修改src/Makefile，将这么一行(在231行的样子)，将$O/lmbench:../scripts/lmbenchbk.ver中的bk.ver去掉，就可以了。
如果一切顺利，编译没有错误，就会出现一些选择提示以对测试进行一个配置并生成配置脚本，后续的测试将使用该配置脚本，在以后测试中也能够直接使用同样的配置多次测试。配置提示除了测试的内存范围（如“MB[default1792]”时，对内存较大的应该避免选择太大值，否则测试时间会很长）和是否Mailresults外，基本上都能够选择缺省值。Lmbench根据配置文档执行任何测试项，在results目录下根据系统类型、系统名和操作系统类型等生成一个子目录，测试结果文档（systemname+序号）存放于该目录下。测试完毕执行makesee可查看到测试结果报告Lmbench的结果及其说明、测试结果及说明。

**相关教程:**

iostat来对linux硬盘IO性能进行检测: