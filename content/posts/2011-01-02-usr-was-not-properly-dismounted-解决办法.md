---
title: /usr was not properly dismounted 解决办法
author: admin
type: post
date: 2011-01-02T06:26:16+00:00
url: /archives/7418
IM_contentdowned:
 - 1
categories:
 - 服务器

---
今日安装好freebsd系统后，就改了一下/etc/rc.conf文件，然后输入reboot重启
重启后发现一个问题，我的用户都无法通过ttyv0-8登陆，无论什么用户，然后没办法，再重启进入单用户模式，df 发现很多区没挂上去，mount -a 挂上/etc/fstab中默认的分区，提示出来了。
/usr was not properly dismounted
/tmp was not properly dismounted
/var was not properly dismounted

然后按照平时的习惯
fsck
fsck -y
fsck -p
结果问题依旧，唉！汗啊！！


**于是上网找方法，找到了这个：**
学习的BSD的教材上，作者明确指出不要用reboot和halt执行重启和关机动作，那样系统不会执行rc.shutdown脚本导致不能在文件系统上设立“清除”标记，下次开机时系统会自动调用FSCK来检查文件系统一的。
呵呵，reboot   halt -p 都不让用呵呵。没办法。只有这样用了
**WARNING: / was not properly dismounted**

我的机器只有在非正常关机之后才会有这种警告.

系统启动后会自动 fsck 的.你等到它查 完了再

> shutdown -h now/halt/reboot/

都不会再出这个信息了.
上面这条命令还是挺管用的，那条错误不报了，但是，我的用户还是登陆不了系统，继续汗啊！！正在解决中！