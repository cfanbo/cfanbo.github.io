---
title: Allowed memory size of 8388608 bytes exhausted解决方法
author: admin
type: post
date: 2010-11-23T13:29:12+00:00
url: /archives/6758
IM_contentdowned:
 - 1
categories:
 - 程序开发

---
出现该错误的原因：

是因为php页面消耗的最大内存默认是为 8M (在PHP的ini件里可以看到) ,如果文件太大 或图片太大 在读取的时候 会发生上述错误。

解决办法：

1，修改 php.ini
将memory_limit由 8M 改成 16M（或更大），重启apache服务

2，在PHP 文件中 加入 ini\_set(”memory\_limit”,”100M”);

注意:为了系统的其它资源的正常使用 请您不要将 memory_limit设置太大，其中-1为不限

3，修改.htaccess 文档（前提是该目录支持.htaccess）
在文档中新增一句：php\_value memory\_limit 16M(或更大)