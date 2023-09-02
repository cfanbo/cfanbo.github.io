---
title: linux下安装opcache扩展(原创)
author: admin
type: post
date: 2013-11-06T06:04:16+00:00
url: /archives/14646
categories:
 - 系统架构
tags:
 - opcache

---
参考： [http://www.php.net/manual/zh/opcache.installation.php](http://www.php.net/manual/zh/opcache.installation.php)

```
wget http://pecl.php.net/get/zendopcache-7.0.2.tgz
tar zxvf zendopcache-7.0.2.tgz
cd zendopcache-7.0.2
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config
make && make install
ls /usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/

```

可以看到 opcache.so 库文件

在PHP.ini添加如下代码

```
[Zend Opcache]
zend_extension = /usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/opcache.so
opcache.memory_consumption=64
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.force_restart_timeout=180
opcache.revalidate_freq=60
opcache.fast_shutdown=1
opcache.enable_cli=1

```

对于上面opcache的各项选项介绍，请参考： [http://www.php.net/manual/zh/opcache.configuration.php](http://www.php.net/manual/zh/opcache.configuration.php)

重启php，通过 phpinfo(); 命令查看

[![php.ini_opcache_extension](http://blog.haohtml.com/wp-content/uploads/2013/11/php.ini_opcache_extension.png)][1]

在phpinfo()信息中, 目前来看有两条信息犹为重要:

Cache hits     (高级缓存命中)Cache misses  (高级缓存未命中)

在这两条信息中即可观察缓存运行情况, 一目了然
高速缓存带来哪些优化呢? 对代码运行有多大帮助?

我们做个测试, 验证一下什么是opcache.

```
echo 'opcache';

```

这是一段非常简单的php代码, 请保存为demo.php文件然后访问, 随意刷新, Cache hits数值会不停地增加, 说明起作用了,
然后你修改代码为:

```
echo 'cache new';

```

再刷新demo.php, 应该可以看到效果, 打印出来的值仍然是opcache, 即源码被缓存了, 它不再解析demo.php文件, 试着不停地刷新, 检测多少秒后才更新.
可设置: opcache.force_restart_timeout=180 的时间来控制更新速度.

这就类似于web项目中的静态文件缓存一下, 比如我们加载一个网页, 浏览器会自动帮我们把jpg, css缓存起来, 唯独php没有缓存, 每次均需要open文件, 解析代码,  执行代码这一过程, 而opcache即可解决这个问题, 代码会被高速缓存起来, 提升访问速度.

xampps环境包已经集成此功能, 不过没有开启, 欢迎大家试用.

个人建议: 开发环境非必要情况下不要开启opcache, 服务器上则建议开启,必竟不是天天更新. 缓存起来有它的历史意义.

**注意：Zend Opcache 与  eaccelerator 相冲突。要安装 Zend Opcache，可能需要先卸载 eaccelerator —— 如果你用了这个加速模块的话。**

**配置参数详解**

 *

**opcache.enable**(默认值:1)



 Zend Optimizer + 的开关, 关闭时代码不再优化.

 *

**opcache.memory_consumption**(默认值:64)



 Zend Optimizer + 共享内存的大小, 总共能够存储多少预编译的 PHP 代码(单位:MB).

 *

**opcache.interned_strings_buffer**(默认值:4)



 Zend Optimizer + 中interned字符串的占内存总量.(单位:MB)

 *

**opcache.max_accelerated_files**(默认值:2000)



 Zend Optimizer + 哈希表中键数量的最大值(一个脚本文件应当是对应一个key的,所以应当就是允许缓存的文件最大数量).这个值实际上是素数列表{ 223, 463, 983, 1979, 3907, 7963, 16229, 32531, 65407, 130987 }中第一个大于设定值的数字.值设定范围: 200 – 100000

 *

**opcache.max_wasted_percentage**(默认值:5)



 “浪费”的内存达到此值对应的百分比,就会发起一个重启调度.

 *

**opcache.use_cwd**(默认值:1)



 开启这条指令, Zend Optimizer + 会自动将当前工作目录的名字追加到脚本键上, 以此消除同名文件间的键值命名冲突.关闭这条指令会提升性能,但是会对已存在的应用造成破坏.

 *

**opcache.validate_timestamps**(默认值:1)



 禁用时, 您必须手动重置Zend Optimizer +或重新启动Web服务器,以使文件系统的更改生效. 检查的频率是由指令 “opcache.revalidate_freq” 控制.

 *

**opcache.revalidate_freq**(默认值:2)



 多久(以秒为单位)检查文件时间戳以改变共享内存的分配.”1″ 表示一秒校验一次, 但是是每个请求一次. “0″ 表示总是在校验.

 *

**opcache.revalidate_path**(默认值:0)



 允许或禁止在 include\_path 中进行文件搜索的优化. 如果文件搜索被禁用而且可以在相同的 include\_path 中找到这个缓存的文件, 文件搜索就不会再进行下去了. 因此,如果 include_path 其它地方有一个同名文件的话, 那就找不到了. 如果这个优化对您的应用有影响,那么应当允许它搜索. 默认情况下,指令是禁止的,这就意味着,优化是处于激活状态的.

 *

**opcache.save_comments**(默认值:1)



 如果禁用,所有的文档注释都从代码中剔除以此减少优化过的代码的大小.禁用 “文档注释” 可能会破坏一些现有的应用和框架(例如: Doctrine, ZF2, PHPUnit).

 *

**opcache.load_comments**(默认值:1)



 如果禁用, PHP文档注释将不会从 SHM(共享内存) 中读取. 尽管”文档注释”还是会被存储(save_comments=1), 但是那些无论如何都用不上的注释就不必被应用读取了.

 *

**opcache.fast_shutdown**(默认值:0)



 如果开启, 一个快速关闭队列用以提速代码. 快速关闭队列并不释放每个已分配的块, 而是让 Zend 引擎内存管理器来干这个活.

 *

**opcache.enable_file_override**(默认值:0)



 允许覆盖文件存在（file_exists等）的优化特性。

 *

**opcache.optimization_level**(默认值:0xffffffff)



 一个位掩码,其中每个位允许或禁用相应的缓存通过.

 *

**opcache.inherited_hack**(默认值:1)



 启用此Hack可以暂时性的解决”can’t redeclare class”错误. Zend Optimizer + 存储着 DECLARE\_CLASS 操作码使用继承的地方(这些是唯一可以被PHP执行的操作码,但是也可能因为优化引起的父类找不到而无法执行).当文件被读取时, Optimizer 会试着通过当前环境绑定被继承的类. 这样做的问题是. DECLARE\_CLASS 的操作码可能不被当前脚本所需要, 如果脚本需要操作码至少完成类的定义操作, 那么它就会无法执行.这指令的默认是禁用的, 这就表示优化是有效的. 该在 php 5.3 以及以上的版中不再被需要, 而且这个设置也不会生效.

 *

**opcache.dups_fix**(默认值:0)



 启用此Hack可以暂时性的解决”can’t redeclare class”错误.

 *

**opcache.blacklist_filename**(默认值:无)



 Zend Optimizer + 黑名单文件的位置.
 Zend Optimizer + 黑名单是一个文本文件包含了那些不能被加速的文件名.文件格式为每行一个文件名.文件名须为一个完整的路径或者紧紧一个文件前缀(如:/var/www/x 屏蔽了 /var/www 文件和目录中所有以 ‘x’ 开始的文件或者目录). 需要屏蔽的文件通常符合下面三个原因中的一个:
 1) 目录包含了自动生成的代码, 如 Smarty 或者 ZFW 的缓存.
 2) 执行加速时代码无法很好的运行, 从而耽误了编译时评估.
 3) 代码触发了一个 Zend Optimizer + 的 Bug

 *

**opcache.max_file_size**(默认值:0)



 通过文件大小屏除大文件的缓存.默认情况下所有的文件都会被缓存.

 *

**opcache.consistency_checks**(默认值:0)



 每 N 次请求检查一次缓存校验.默认值0表示检查被禁用了.由于计算校验值有损性能,这个指令应当紧紧在开发调试的时候开启.

 *

**opcache.force_restart_timeout**(默认值:180)



 从缓存不被访问后,等待多久后(单位为秒)调度重启.Zend Optimizer + 依托此指令来确定一个进程可能在处理过程中出现问题的情况.这段时间(等待时间)过后, 假设 Zend Optimizer + 发生了一些问题, 并开始干掉那些仍然持有预防重启锁的进程.当这些发生时, 如果日志的级别是3级或以上, 一个 “killed locker” 的错误就会被记录到 Apache 的日志中.

 *

**opcache.error_log**(默认值:无)



 Zend Optimizer + 的错误日志文件名.留空表示使用标准错误输出(stderr).

 *

**opcache.log_verbosity_level**(默认值:1)



 将错误信息都导向 Web 服务器日志.默认的只有致命错误(level 0) 或者错误(level 1)才会被记录.你也可以允许警告(level 2),提示消息(level 3) 或者 调试消息(level 4)被记录下来.

 *

**opcache.preferred_memory_model**(默认值:无)



 内存共享的首选后台.留空则是让系统选择.

 *

**opcache.protect_memory**(默认值:0)



 防止共享内存在脚本执行期间被意外写入, 仅用于内部调试.

 *

**opcache.mmap_base**(默认值:无)



 共享内存段映射基础(仅适用于Windows).所有的PHP进程必须映射到相同的共享内存地址空间.该指令用于手动修复 “Unable to reattach to base address” 错误.


 名字

 默认

 可修改范围

 含义

 opcache.enable

 “1”

 PHP_INI_ALL

 是否启用opcache

 opcache.enable_cli

 “0”

 PHP_INI_SYSTEM

 是否在CLI（即命令行时）启用opcache

 opcache.memory_consumption

 “64”

 PHP_INI_SYSTEM

 为opcache分配多少共享内存，单位M

 opcache.interned_strings_buffer

 “4”

 PHP_INI_SYSTEM
 interned string的内存大小
 opcache.max_accelerated_files

 “2000”

 PHP_INI_SYSTEM


最大缓存的文件数目。


实际上这个值会使用第一个大于你配置的数字的下列素数


{ 223, 463, 983, 1979, 3907, 7963, 16229, 32531, 65407, 130987 }，

如你将该值指定为400，则实际上该值为463.

 opcache.max_wasted_percentage

 “5”

 PHP_INI_SYSTEM

 opcache.use_cwd

 “1”

 PHP_INI_SYSTEM


如果置为1，则将当前路径加入到文件key中，


以避免可能产生的同文件名的文件key冲突


 opcache.validate_timestamps

 “1”

 PHP_INI_ALL


如果置为1，则OPCACHE会自动检测文件的时间戳


（检测周期为revalidate_freq),


并根据文件的时间戳来更新opcode,如果置为0，


则只能手动去重启opcache或


重启webserver以使更新后的php文件生效


 opcache.revalidate_freq

 “2”

 PHP_INI_ALL


opcache自动检测文件是否更新的周期，单位秒。


如果是0，则每次请求时opcache都要进行检测。


当validate_timestamps为0时，本指令无效。


 opcache.revalidate_path

 “0”

 PHP_INI_ALL

 opcache.save_comments

 “1”

 PHP_INI_SYSTEM

 是否保存文件中的注释

 opcache.load_comments

 “1”

 PHP_INI_ALL


是否load comments，与save_comments联合起来使用，


如果该值为0，则即使save_comments为1，


那么php脚本中的comments也是不使用的


 opcache.fast_shutdown

 “0”

 PHP_INI_SYSTEM


是否打开快速关闭，


打开时可使php在request shutdown时回收内存快


 opcache.enable_file_override

 “0”

 PHP_INI_SYSTEM


如果置为1，则每次调用file_exist() is_file() is_readable()函数时，


opcache将要检查该文件是否被cache了，


这样增加了检查存在性和可读性的开销，


但避免了当validate_timestamps为disable时返回错误文件状态的风险。


 opcache.optimization_level

 “0xffffffff”

 PHP_INI_SYSTEM

 运行时控制优化的掩码（干什么的？）

 opcache.inherited_hack

 “1”

 PHP_INI_SYSTEM

 5.3以前使用。5.3后废弃

 opcache.dups_fix

 “0”

 PHP_INI_ALL

 为解决“cannot redecllare class” 时，可将其置为1

 opcache.blacklist_filename

 “”

 PHP_INI_SYSTEM


设置黑名单文件，符合黑名单文件中定义的php文件将不被opcache。黑名单文件的例子如下：


```
; Matches a specific file. /var/www/broken.php ; A prefix that matches all files starting with x. /var/www/x ; A wildcard match. /var/www/*-broken.php
```

```
一行为一条规则，支持通配符，注释以分号开头
```

 opcache.max_file_size

 “0”

 PHP_INI_SYSTEM

 被cache的文件的最大size,单位bytes。0表示不限

 opcache.consistency_checks

 “0”

 PHP_INI_ALL


如果置为N，N非零，则opcache会每N个请求核实一下cache的检验和。


这会损害性能，应该只在debug时使用


 opcache.force_restart_timeout

 “180”

 PHP_INI_SYSTEM

 如果opcache处于非active状态，当N秒后opcache将自动重启

 opcache.error_log

 “”

 PHP_INI_SYSTEM

 opcache自身的errorlog文件路径，为空时则使用stderr

 opcache.log_verbosity_level

 “1”

 PHP_INI_SYSTEM

 日志记录level，默认只有fatal error和error

 opcache.preferred_memory_model

 “”

 PHP_INI_SYSTEM


opcache首选使用的内存模型，为空时会选择最适当的模型。


常用的有，mmap shm posix 和win32


 opcache.protect_memory

 “0”

 PHP_INI_SYSTEM


运行php脚本时保护共享内存防止意外的写入。


只对debug时有用。


 opcache.mmap_base
 `NULL`
 PHP_INI_SYSTEM


 相关文章：

 # [使用 Zend Opcache 加速 PHP](http://cnzhx.net/blog/zendopcache-accelerate-php/)

 [1]: http://blog.haohtml.com/wp-content/uploads/2013/11/php.ini_opcache_extension.png