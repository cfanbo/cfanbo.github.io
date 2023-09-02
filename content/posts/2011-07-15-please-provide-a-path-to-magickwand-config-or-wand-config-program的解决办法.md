---
title: Please provide a path to MagickWand-config or Wand-config program的解决办法
author: admin
type: post
date: 2011-07-15T17:14:51+00:00
url: /archives/10454
IM_contentdowned:
 - 1
categories:
 - 服务器

---
今天在安装lnmp的时候,发现在安装imagick-3.0.1.tgz时,执行

>

```
./configure --with-php-config=/usr/local/php/bin/php-config
```

```
的时候,提示以下错误:
```

```
checking for PHP includes… -I/usr/include/php -I/usr/include/php/main -I/usr/include/php/TSRM -I/usr/include/php/Zend -I/usr/include/php/ext
checking for PHP extension directory… /usr/lib64/php/modules
checking for PHP installed headers prefix… /usr/include/php
checking for re2c… no
configure: WARNING: You will need re2c 0.9.11 or later if you want to regenerate PHP parsers.
checking for gawk… gawk
checking whether to enable the magickwand extension… no
configure: error: not found. Please provide a path to MagickWand-config or Wand-config program.
```

**解决办法:**

> yum install ImageMagick-devel

然后重新上面的./configure命令即可.