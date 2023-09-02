---
title: 'configure: error: newly created file is older than distributed files!'
author: admin
type: post
date: 2011-04-06T06:08:13+00:00
url: /archives/9021
IM_contentdowned:
 - 1
categories:
 - 服务器

---
在linux下安装软件包的时候,有时候提示

> configure: error: newly created file is older than distributed files!
> Check your system clock

出现此编译错误，请检查你的系统时间是否设置有误。。。

查看硬件日期时间

> hwclock -show

linux是每隔一段时间将系统时间写入 硬件bois的 如果刚设置完了就关机,开机后时间还是等于没有设置

> \# date -s 991128

Sun Nov 28 00:00:00 CST 1999

> 实例：设置时间伟2008年8月8号12:00
>
> \# date -s “2008-08-08 12:00:00″

修改完后,记得输入:

> clock -w

把系统时间写入CMOS即可