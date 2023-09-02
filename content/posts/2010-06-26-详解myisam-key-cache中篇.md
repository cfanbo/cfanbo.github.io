---
title: 详解MyISAM Key Cache(中篇)
author: admin
type: post
date: 2010-06-26T16:57:52+00:00
url: /archives/4108
IM_contentdowned:
 - 1
categories:
 - MySQL

---
在[前篇][1]中 介绍了Key Cache的基本机制，并且介绍了Key Cache的LRU算法。作为对LRU算法的改进，MyISAM还提供了另一个缓存算法：“Midpoint Insertion Strategy”。本文将重点介绍该算法的原理和配置。

1. 相关参数

该策略涉及的参数有：[key\_cache\_division_limit][2]、[key\_cache\_age_threshold][3]

2. 原理介绍

(1) 该策略将前面的LRU队列（LRU Chain）分成两部分，hot sub-chain和warm sub-chain。并根据参数key\_cache\_division\_limit划分，总保持warm sub-chain在这个百分比以上。默认情况key\_cache\_division\_limit是100，所以默认时候只有warm sub-chain,即LRU Chain。
(注：Multiple Key cache情况，每个key cache都有对应的key\_cache\_division_limit值)

(2) 在warm sub-chain中的某个block如果被访问（Access）次数超过某个值时候，就将该block放到hot sub-chain的底部。

(3) 在hot sub-chain中的block会随着每一次的hit调整位置，hit越多，越接近底部。在顶部停留时间过长就会被降级到warm sub-chain中，而且是warm sub-chain的顶部（很可能很快就会被移出key cache）。

(4) Hot sub-chain中的顶部的block停留时间超过一个阈值后就会被降级到warm sub-chain。这个阈值由参数key\_cache\_age\_threshold决定。具体的计算方法是：设N为key cache中的block个数，如果在最近的（N*key\_cache\_age\_threshold/100）次访问中，key cache顶部的block仍然没有被访问到，那么就会被移到warm sub-chain的顶部。

(5) 默认情况key\_cache\_division_limit = 100，这时只有只有一个Chain，所以不使用该策略。即退化的Midpoint Insertion Strategy是LRU算法。

3. 如何使用Midpoint Insertion Strategy

我们可以通过配置[key\_cache\_division_limit][2]、[key\_cache\_age_threshold][3]的 值来控制。参数[key\_cache\_division_limit][2]控 制了Key Cache Chain中warm sub-chain的百分比，如果你的Index Block中明显有30%是非常Hot（较之其他的Block更加被常常访问），那么你可以设置你的warm sub-chain长度为70%，剩余30%作为hot sub-chain。参数[key\_cache\_age_threshold][3]定 义了warm sub-chain中的block被移除的机制（参照前文介绍）。

（未完待续）

 [1]: http://www.orczhou.com/index.php/2010/01/myisam-key-buffer-1/
 [2]: http://dev.mysql.com/doc/refman/5.0/en/server-system-variables.html#sysvar_key_cache_division_limit
 [3]: http://dev.mysql.com/doc/refman/5.0/en/server-system-variables.html#sysvar_key_cache_age_threshold