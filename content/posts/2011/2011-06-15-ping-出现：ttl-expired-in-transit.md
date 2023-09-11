---
title: Ping 出现：TTL expired in transit
author: admin
type: post
date: 2011-06-15T09:38:17+00:00
url: /archives/9853
IM_contentdowned:
 - 1
categories:
 - 其它

---
今天发现电信送的一条固定IP地址出现问题,查确认属于是他们私自更改了我们的IP地址.电话通知电信大客经理…在下午接通知，已经改好！
我觉得还是自已测试一下，不能太相信别人的话，因此我通知他们稍等下．
１、我先PING了一下IP地址，结果发现：

> C:\Documents and Settings\xm_dengwh>ping 218.xxx.xxx.xxx (这里是我们的IP地址)
>
> Pinging 218.xx.xx.xx with 32 bytes of data:
>
> Reply from 218.85.151.173: TTL expired in transit.
> Reply from 218.85.151.173: TTL expired in transit.
> Reply from 218.85.151.173: TTL expired in transit.
> Reply from 218.85.151.173: TTL expired in transit.

需要注意的是: 我的IP地址:218.xxx.xxx.xxx和218.85.151.173不同.

不是正常的TTL返回时间，从提示来看应该是TTL耗尽了，为什么ＴＴＬ会耗尽呢？难道是路由器配置错误，形成环路了使数据包不停的在两个路由器之间传送，使ＴＴＬ耗尽？为了证实我的猜想,我觉定使用tracert看一下所经过的路由器情况.

> C:\Documents and Settings\xm_dengwh>tracert 218.85.xx.xx
>
> Tracing route to mx2.bestcheer.com [218.85.xxx.xxx]
> over a maximum of 30 hops:
>
> 1 10 ms 1 ms 1 ms 59.xx.xx.xx
> 2 1 ms 2 ms 1 ms 61.154.238.102
> 3 \* \* * Request timed out.
> 4 1 ms 3 ms 1 ms 218.85.151.173
> 5 \* \* * Request timed out.
> 6 2 ms 2 ms 1 ms 218.85.151.173
> 7 \* \* * Request timed out.
> 8 2 ms 2 ms 2 ms 218.85.151.173
> 9 \* \* * Request timed out.
> 10 * 2 ms 2 ms 218.85.151.173
> 11 \* \* * Request timed out.
> 12 3 ms 3 ms 3 ms 218.85.151.173

从结果来看,应该是第3hop转数据包到第4hop(218.85.151.173)上,然后又转第3hop上,因此数据在第3路由器与第4个路由器之间造成循环使数据不停的两个路由器之间转发.
马上通知电信公司,把路由配置错误,造成数据包循环的情况告诉他们,20分钟后问题解决.