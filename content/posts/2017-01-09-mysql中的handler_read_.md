---
title: mysql中的handler_read_%
author: admin
type: post
date: 2017-01-09T05:58:36+00:00
url: /archives/17358
categories:
 - MySQL

---

01. mysql> show status like ‘handler_read_%’;

02. +———————–+——-+

03. | Variable_name | Value |

04. +———————–+——-+

05. | Handler_read_first | 1 |

06. | Handler_read_key | 1 |

07. | Handler_read_last | 0 |

08. | Handler_read_next | 0 |

09. | Handler_read_prev | 0 |

10. | Handler_read_rnd | 0 |
11. | Handler_read_rnd_next | 21 |
12. +———————–+——-+

13. 7 rows in set (0.01 sec)


如上所示，mysql中关于read的计数器，有7个。他们的数值对于系统的状况的了解，对于系统的调优都十分重要。我们应该理解他们的含义。本文是自己的一些理解。
首先7个计数器，我们应该分为两部分：
1）对索引读的计数器：前面的5个都是对索引读情况的计数器，
Handler\_read\_first：是指读索引的第一项（的次数）；
Handler\_read\_key：是指读索引的某一项（的次数）；
Handler\_read\_next：是指读索引的下一项（的次数）；
Handler\_read\_last：是指读索引的最后第一项（的次数）；
Handler\_read\_prev：是指读索引的前一项（的次数）；
5者应该有四种组合：
1. Handler\_read\_first 和 Handler\_read\_next 组合应该是索引覆盖扫描
2. Handler\_read\_key 基于索引取值
3. Handler\_read\_key 和 Handler\_read\_next 组合应该是索引范围扫描
4. Handler\_read\_last 和 Handler\_read\_prev 组合应该是索引范围扫描（orde by desc）

2）对数据文件的计数器：后面的2个都是对数据文件读情况的计数器，
Handler\_read\_rnd:
The number of requests to read a row based on a fixed position. This value is high if you are doing a lot of queries that require sorting of the result. You probably have a lot of queries that require MySQL to scan entire tables or you have joins that do not use keys properly.

Handler\_read\_rnd_next

```
The number of requests to read the next row in the data file. This value is high if you are doing a lot of
table scans. Generally this suggests that your tables are not properly indexed or that your queries are
not written to take advantage of the indexes you have.

这里很重要的一点要理解：索引项之间都是有顺序的，所以才有first, last, next, prev等等，所以前面的5个都是对索引读情况
的计数器，而后面的2个是对数据文件的读情况的计数器。

很显然的一点：
后面的2个 Handler_read_rnd 和 Handler_read_rnd_next 是越低越好，如果很高，应该进行索引相关的调优。而Handler_read_key的数值
肯定是越高越好，越高代表使用索引读很高。其他的计数器，要具体情况具体分析
```

http://blog.chinaunix.net/uid-25909722-id-4378355.html