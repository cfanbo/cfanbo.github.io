---
title: 用shell分析nginx日志里的访问最多的IP地址
author: admin
type: post
date: 2015-07-30T09:59:08+00:00
url: /archives/15914
categories:
 - 服务器

---

```
# $3的位置是IP地址，可按情况修改，如：
# [30/Sep/2012:19:14:47 +0800] 110.75.176.58 www.example.com "GET / HTTP/1.1" 200 3629 "-" "Yahoo! Slurp China"
cat nginx.log | awk '{print $3}' | sort | uniq -c | sort -nr | less

#输出：
#    120 189.17.37.109
#     96 12.15.61.22
#     95 12.20.29.33
#     。。。 。。。
```