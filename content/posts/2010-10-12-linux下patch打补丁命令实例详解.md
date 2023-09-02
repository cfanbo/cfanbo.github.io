---
title: Linux下patch打补丁命令实例详解
author: admin
type: post
date: 2010-10-12T01:39:53+00:00
url: /archives/5980
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - 补丁
 - Linux
 - patch

---
**linux下patch命令使用详解—linux打补丁命令**

**功能说明：**

****修补文件。

**语　　法：**

******p****atch** \[-bceEflnNRstTuvZ\]\[-B <备份字首字符串>\]\[-d <工作目录>\]\[-D <标示符号>\]\[-F <监别列数>\]\[-g <控制数值>\]\[-i <修补文件>\]\[-o <输出文件>\]\[-p <剥离层级>\]\[-r <拒绝文件>\]\[-V <备份方式>\]\[-Y <备份字首字符串>\]\[-z <备份字尾字符串>\]\[–backup-if　　 -mismatch\]\[–binary\]\[–help\]\[–nobackup-if-mismatch\]\[–verbose\][原始文件 <修补文件>] 或 patch [-p <剥离层级>] < [修补文件]

**补充说明：**

******patch**指令让用户利用设置修补文件的方式，修改，更新原始文件。倘若一次仅修改一个文件，可直接在指令列中下达指令依序执行。如果配合修补文件的方式则能一次修补大批文件，这也是Linux系统核心的升级方法之一。

**参　　数：**
-b或–backup 　备份每一个原始文件。
-B<备份字首字符串>或–prefix=<备份字首字符串> 　设置文件备份时，附加在文件名称前面的字首字符串，该字符串可以是路径名称。
-c或–context 　把修补数据解译成关联性的差异。
-d<工作目录>或–directory=<工作目录> 　设置工作目录。


-D<标示符号>或–ifdef=<标示符号> 　用指定的符号把改变的地方标示出来。
-e或–ed 　把修补数据解译成ed指令可用的叙述文件。
-E或–remove-empty-files 　若修补过后输出的文件其内容是一片空白，则移除该文件。
-f或–force 　此参数的效果和指定-t参数类似，但会假设修补数据的版本为新　版本。
-F<监别列数>或–fuzz<监别列数> 　设置监别列数的最大值。
-g<控制数值>或–get=<控制数值> 　设置以RSC或SCCS控制修补作业。
-i<修补文件>或–input=<修补文件> 　读取指定的修补问家你。
-l或–ignore-whitespace 　忽略修补数据与输入数据的跳格，空格字符。
-n或–normal 　把修补数据解译成一般性的差异。
-N或–forward 　忽略修补的数据较原始文件的版本更旧，或该版本的修补数据已使　用过。
-o<输出文件>或–output=<输出文件> 　设置输出文件的名称，修补过的文件会以该名称存放。
-p<剥离层级>或–strip=<剥离层级> 　设置欲剥离几层路径名称。
-f<拒绝文件>或–reject-file=<拒绝文件> 　设置保存拒绝修补相关信息的文件名称，预设的文件名称为.rej。
-R或–reverse 　假设修补数据是由新旧文件交换位置而产生。
-s或–quiet或–silent 　不显示指令执行过程，除非发生错误。
-t或–batch 　自动略过错误，不询问任何问题。
-T或–set-time 　此参数的效果和指定-Z参数类似，但以本地时间为主。
-u或–unified 　把修补数据解译成一致化的差异。
-v或–version 　显示版本信息。
-V<备份方式>或–version-control=<备份方式> 　用-b参数备份目标文件后，备份文件的字尾会被加上一个备份字符串，这个字符串不仅可用-z参数变更，当使用-V参数指定不同备份方式时，也会产生不同字尾的备份字符串。
-Y<备份字首字符串>或–basename-prefix=–<备份字首字符串> 　设置文件备份时，附加在文件基本名称开头的字首字符串。
-z<备份字尾字符串>或–suffix=<备份字尾字符串> 　此参数的效果和指定-B参数类似，差别在于修补作业使用的路径与文件名若为src/linux/fs/super.c，加上backup/字符串后，文件super.c会备份于/src/linux/fs/backup目录里。
-Z或–set-utc 　把修补过的文件更改，存取时间设为UTC。
–backup-if-mismatch 　在修补数据不完全吻合，且没有刻意指定要备份文件时，才备份文件。
–binary 　以二进制模式读写数据，而不通过标准输出设备。
–help 　在线帮助。
–nobackup-if-mismatch 　在修补数据不完全吻合，且没有刻意指定要备份文件时，不要备份文件。
–verbose 　详细显示指令的执行过程。

patch，是打补丁的命令，有很多用法，见帮助#man patch

> patch -p0       (“p”指的是路径，后面的数字表示去掉路径的第几部分。0，表示不去掉，为全路径)
> patch -p1       (“p”后面的数字1，表示去掉前第一个路径)

fetch http://people.freebsd.org/~delphij/misc/patch-bge-releng62
fetch http://people.freebsd.org/~delphij/misc/patch-bce-watchdog-rewritecd /sys/dev/bge
fetch …
patch -p0 < …fetch http://people.freebsd.org/~delphij/misc/patch-tcp_auto_buf-20061212-RELENG_6.diffpatch -p < patch-tcp_auto_buf-20061212-RELENG_6.diff
也可以把文件中的目录全改成系统已在的目录如/usr/src/sys…..注意：
１，确认目录
然后确认目录，如不在默认目录下，就写下要打补丁的当前绝对目录。如/usr/src/sys/dev/bge/if_bce.c２，P的使用可以使用不带数字的参数。
patch 后的软件安装

telnetd服务器的问题及补丁 在当前FreeBSD所有版本中，也就是FreeBSD 5.0、FreeBSD 4.3、FreeBSD 4.2、FreeBSD 4.1.1、FreeBSD 4.1、FreeBSD 4.0、FreeBSD 3.x、FreeBSD 2.x的版本，其telnetd守护进程中存在一个致命的缓冲区溢出漏洞，该问题是由于telnetd在处理telnet协议选项的函数中没有进行有效的边界检查，当使用某些选项（’AYT’）时，可能发生缓冲区溢出。这会导致远程root级别的安全威胁。