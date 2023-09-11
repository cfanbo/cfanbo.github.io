---
title: Linux环境下PCRE库的使用
author: admin
type: post
date: 2011-04-05T04:20:21+00:00
url: /archives/8982
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - Linux
 - pcre

---
今天下载了PCRE的正则表达式库，应用在Linux环境下的C语言编程中。

调用方法：

1.下载PCRE库：[**ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/**][1]，版本是7.8；

2.解压后执行configure，而后make，make install，可配置后动态链接库；

3.写了个测试的例子：

01. #include
02. #include
03. int main()

04. {

05. pcre *re;

06. const char *error;

07. int erroffset;

08. int rc;

09. int ovector[30];

10. re = pcre_compile(“some”, 0, &error, &erroffset, NULL);

11. rc = pcre_exec(re, NULL, “some string”, 11, 0, 0, ovector, 30);

12. printf(“%d\n”, rc);

13. return 0;

14. }


4.gcc -o test test.c -lpcre

5../test

6.参考文档：

上述有很多细节要搞清楚，还需要仔细研究。

出自：



 [1]: ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/