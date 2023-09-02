---
title: 'ld: symbol(s) not found for architecture x86_64 clang: error: linker command failed with exit code 1 (use -v to see invocation) make: *** [sapi/cli/php] Error 1'
author: admin
type: post
date: 2015-12-10T22:28:47+00:00
url: /archives/16296
categories:
 - 程序开发

---
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: \*** [sapi/cli/php] Error 1

解决办法：

make -lstdc++

即可。

在mac中安装php7遇到了此问题，但用此方法无法解决。