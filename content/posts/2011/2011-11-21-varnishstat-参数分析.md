---
title: varnishstat 参数分析
author: admin
type: post
date: 2011-11-21T07:05:27+00:00
url: /archives/12064
IM_contentdowned:
 - 1
categories:
 - 系统架构

---
Hitrate ratio由三个数字组成，第一个数字范围0-10，第二个数字范围0-100，第三个数字范围0-1000。

分别表示过去N秒内的Hitrate avg。上图由于我是刚打开varnishstat，因此三个数字都是4，表示过去4秒内的平均hitrate，如果打开的时间足够长，以上三个数字就会逐渐变成**10，100，1000**。

Hitrate avg里的内容是命中率，需要乘以100转换成百分比，例如上图表示命中率为99.23%

接着往下看，三列数据分别表示实时数据，每秒平均值，自启动以来每秒平均值。有些参数是没有后两列的，这是因为这些值都有固定变动范围，例如N work threads，只会在0到最大值（我设的是200）之间变动，搞每秒平均值意义不大（我猜）。

**以下指标需要重点关注一下：**

Client connections accepted: （每秒处理连接数）。
Client requests received：经验表明connection:request=1:10左右时比较理想，比这个数大很多或者小很多都是不好的。代表到目前为止，浏览器向反向代理服务器发送的HTTP请求累积次数，由于可能使用长连接，所以它可能会大于上边的Client connections accepted。
Backend connections failures：这个数应该尽可能小，没有就最好，多的话就要看看backend指向的服务是否有问题了。
N struct object：当前被cache的条目。
N worker threads：当前工作线程数。
N worker threads created：创建了多少线程(should be close to the number you are running now) 。
N worker threads not created ：最好是0，表示varnish尝试创建线程但失败。
N worker threads limited ：由于线程上限限制或者线程池反应延迟导致不能成功创建的线程数，越小越好。
N overflowed work requests ：进入等待队列的请求数，越小越好。
N dropped work requests：队列被塞满后扔掉的请求，这个最好不要有。
N LRU nuked objects：由于cache空间满而不得不扔掉的cache条目，如果这个数字是0，就没必要增加cache的大小了。
n_expired ：由于cache时间超时而被扔掉的cache条目，具体要看你的ttl设多大了。
Cache hits: 代表在这些请求中，反向代理服务器在缓存区中查找并且命中缓存的次数。
Cache misses: 代表在这些请求中，反向代理服务器在缓存区中查找但是没有命中缓存的次数。
N expired objects: 代表过期的缓存内容个数
N LRU nuked objects: 由于cache空间满而不得不扔掉的cache条目，如果这个数字是0，就没必要增加ache的大小了。
N LRU moved objects: 代表被淘汰的缓存内容个数
Total header bytes: 代表缓存区中所有缓存内容的HTTP头信息长度
Total body bytes: 代表缓存区中所有缓存内容的正文长度

**其他用法：**
使用-1参数能看到所有的性能数据，例如：

> varnishstat -1 -n /workdir/

使用varnishstat -f field1,field2,field3,…,fieldN能查看想要关心的参数。

更多用法请用万能的-h参数。