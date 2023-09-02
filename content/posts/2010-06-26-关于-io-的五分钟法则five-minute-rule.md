---
title: 关于 I/O 的五分钟法则(Five-Minute Rule)
author: admin
type: post
date: 2010-06-26T07:28:28+00:00
url: /archives/3997
IM_contentdowned:
 - 1
categories:
 - 系统架构

---
去年在对 SSD 做调查的时候就关注过这个[五分钟法则][1]，今天又发现了[这 篇文章][2]的修订版(为了纪念 Jim Gray），这个话题倒是可以简单介绍一下，对架构师衡量 I/O 能力、Cache 评估和做硬件选型还是会有一些帮助的。

在 1987 年，[Jim Gray][3] 与 Gianfranco Putzolu 发表了这个”五分钟法则”的观点，简而言之，如果一条记录频繁被访问，就应该放到内存里，否则的话就应该待在硬盘上按需要再访问。这个临界点就是五分钟。 看上去像一条经验性的法则，实际上五分钟的评估标准是根据投入成本判断的，根据当时的硬件发展水准，在内存中保持 1KB 的数据成本相当于硬盘中存储同样大小数据 400 秒的开销(接近五分钟)。这个法则在 1997 年左右的时候进行过一次回顾，证实了五分钟法则依然有效（硬盘、内存实际上没有质的飞跃)，而这次的回顾则是针对 SSD 这个”新的旧硬件”可能带来的影响。

[![](http://blog.haohtml.com/wp-content/uploads/2010/06/Five-Minute-Rule.gif)][4]

随着闪存时代的来临，五分钟法则一分为二：是把 SSD 当成较慢的内存（extended buffer pool ）使用还是当成较快的硬盘（extended disk）使用。小内存页在内存和闪存之间的移动对比大内存页在闪存和磁盘之间的移动。在这个法则首次提出的 20 年之后，在闪存时代，5 分钟法则依然有效，只不过适合更大的内存页(适合 64KB 的页，这个页大小的变化恰恰体现了计算机硬件工艺的发展，以及带宽、延时)。

根据数据结构和数据特点的不同，对于文件系统来说, 操作系统倾向于将 SSD 当作瞬时内存(cache)来使用。而对于数据库，倾向于将 SSD 当作一致性存储来用。

这是一篇非常重要的文章(所以，建议读一下原文），其中对于数据库一节的描述尤其有趣（针对 DB 也有个五分钟)。限于篇幅，就不罗嗦了。

–EOF–

本文来自： [http://www.dbanotes.net/arch/five-minute_rule.html](http://www.dbanotes.net/arch/five-minute_rule.html)

 [1]: http://queue.acm.org/detail.cfm?id=1413264
 [2]: http://cacm.acm.org/magazines/2009/7/32091-the-five-minute-rule-20-years-later/fulltext#R15
 [3]: http://en.wikipedia.org/wiki/Jim_Gray_%28computer_scientist%29
 [4]: http://blog.haohtml.com/wp-content/uploads/2010/06/Five-Minute-Rule.gif