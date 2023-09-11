---
title: 使用pt-stalk诊断MySQL问题
author: admin
type: post
date: 2012-07-22T05:02:49+00:00
url: /archives/13219
categories:
 - MySQL
tags:
 - mysql

---
在MySQL服务器出现短暂(5~30秒)的性能波动的时候，一般的性能监控工具都很难抓住故障现场，也就很难收集对应较细粒度的诊断信息。另外，如果这种波动出现的频率很低，例如几天才一次，我们也很难人为的抓住现场，收集数据。这正是pt-stalk所解决的问题。

pt-stalk是 [Percona-Toolkit](http://www.percona.com/software/percona-toolkit/) 的一部分(其前身是[Aspersa][1]的一部分)。安装Percona-Toolkit后，可以通过man pt-stalk了解如何使用该工具，本文的介绍是man pt-stalk的一个子集，强烈建议直接阅读man pt-stalk。额外的，本文将提供pt-stalk示例命令可供参考。

**1. 使用pt-stalk**

pt-stalk –collect-tcpdump –function status \

–variable Threads_connected –threshold 2500 \

–daemonize — –user=root –password=YOURPASSWORD

上面的命令表示，让pt-stalk后台运行(–daemonize)，并监视SHOW GLOBAL STATUS中的Threads_connected状态值，如果该值超过2500，则触发收集主机和MySQL的性能、状态信息。pt-stalk会每隔一秒检查一次状态值，如果连续5次满足触发条件，则开始收集。

–collect-tcpdump表示除了收集基本信息外，还将额外使用tcpdump收集当时的网络包，类似的还可以使用–collect-gdb等。

**2. pt-stalk如何连接MySQL**

在上面的命令中参数，”– –user=root –password=YOURPASSWORD”表示，将使用”–“后面的所有参数用于mysql和mysqladmin命令，所以这里确保你给出正确的用户名和密码。下面是man pt-stalk中给出的语法：

SYNOPSIS

Usage: pt-stalk [OPTIONS] [– MYSQL OPTIONS]

看到前面的[OPTIONS]是pt-stalk使用的参数，[– MYSQL OPTIONS]是mysql和mysqladmin使用的参数。

**3. pt-stalk的工作状态**

pt-stalk是一个后台程序，默认我们可以通过文件/var/log/pt-stalk.log，查看pt-stalk的运行状态：

tail -f /var/log/pt-stalk.log

2012_06_05_00_00_35 Check results: Threads_connected=1641, matched=no

2012_06_05_00_00_36 Check results: Threads_connected=1641, matched=no

2012_06_05_00_00_37 Check results: Threads_connected=1641, matched=no

2012_06_05_00_00_38 Check results: Threads_connected=1641, matched=no

2012_06_05_00_00_39 Check results: Threads_connected=1641, matched=no

2012_06_05_00_00_40 Check results: Threads_connected=1641, matched=no

2012_06_05_00_00_41 Check results: Threads_connected=1641, matched=no

你还可以通过参数–log指定一个你希望的log目录和文件。

**4. pt-stalk收集的性能和状态数据**

默认pt-stalk将收集的数据放在目录/var/lib/pt-stalk下，你可以使用参数–dest指定你希望的目录。下面是一个pt-stalk触发收集后的数据文件：

[![pt-stalk数据](http://farm9.staticflickr.com/8146/7169024725_ec0d182348_z.jpg)][2]

这些数据都是原始数据，我们可以根据这些来分析当时MySQL或者主机是否有异常。

**5. pt-stalk的触发条件**

在上面的示例中触发参数是：”–function status –variable Threads\_connected –threshold 2500″，表示MySQL状态值Threads\_connected超过2500时触发数据收集。常用的触发条件还可以使用Threads_running等。

另外还可以使用SHOW PROCESSLIST的中的结果触发，例如”–function processlist –variable State –match statistics –threshold 10″表示，show processlist中State列的值为statistics的线程数超过10则触发收集。

**6. 一些其他有用的参数**

–iterations：该参数指定pt-stalk在收集几次故障现场后就退出。默认pt-stalk会一直运行

–run-time：触发收集后，该参数指定收集**多长时间**的数据。默认是30秒

–sleep：为防止一直触发收集数据，该参数指定在某次触发后，必须sleep一段时候才继续观察并触发收集。默认是300秒

–interval：默认情况pt-stalk会每隔一秒检查一次状态数据，判断是否需要触发收集。该参数指定间隔时间，默认是1秒。

–cycles：默认情况pt-stalk只有连续观察到五次状态值满足触发条件时，才触发收集。该参数控制，需要连续几次满足条件，收集被触发，默认是5次。

参考文献：man pt-stalk;man percona-toolkit

摘自：

 [1]: http://code.google.com/p/aspersa/
 [2]: http://www.flickr.com/photos/26825745@N06/7169024725/ "pt-stalk数据 by orczhou, on Flickr"