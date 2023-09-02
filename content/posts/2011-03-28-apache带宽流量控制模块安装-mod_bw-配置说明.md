---
title: Apache带宽流量控制模块安装 mod_bw 配置说明
author: admin
type: post
date: 2011-03-28T13:31:13+00:00
url: /archives/8205
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - apache
 - mod_bw
 - 流量控制

---
注：这个模块在win32平台下好像不是太稳定的，有的文件可以限制，而有的文件则不行．用的是windows2003操作系统，如果有同样的问题的请，请在这里留言，请注明一下系统类型.

官方网站：http://modules.apache.org,在里面找到一个  Bandwidth Module  的 module，模块说明文档， [点击查看mod_bw-0.说明文档9.0](http://blog.haohtml.com/wp-content/uploads/2011/03/mod_bw-0.9.txt)作者的官方网站 [http://ivn.cl/apache/](http://ivn.cl/apache/)(Win32: [http://ivn.cl/files/dlls/mod_bw-0.91-2.2.14/mod_bw.dll](http://ivn.cl/files/dlls/mod_bw-0.91-2.2.14/mod_bw.dll) 可以下载到。

**Installing step:**
1. 将 mod_bw.dll 放到安装 apache 资料夹下的 modules
2. 编辑 httpd.conf，将  LoadModule bw\_module modules/mod\_bw.dll  加入
3. 重开 apache
4. 查看 phpinfo() 里是否有mod_bw


Configuration Directives:
**1 – BandWidthModule [On|Off]**
这个 module 预设是关闭的，要将他开启才能够使用。

Example:
BandWidthModule On

**2 – ForceBandWidthModule [On|Off]**
这个 module 预设不会过滤每个需求。如果您开启他，他将处理过滤每个需求。

Example :

(正常的使用下，仅会过滤 text/html test/plain)AddOutputFilterByType MOD_BW text/html text/plain

(开启的状况下)ForceBandWidthModule On

**3 – BandWidth [From] [bytes/s]**
这边有两个参数。
From 是限制来源的位置，也就是该位置受限制。他可以是完整的 hostname、网域名称或 IP。可搭配遮罩使用，例如 192.168.0.0/24 or 192.168.0.0/255.255.255.0 。另一个参数是限制的速率，以 bytes 每秒为单位；假如为 0，则不受限制。

Example :
BandWidth localhost 10240
BandWidth 192.168.218.5 0

( 依照设定的先后顺序为排序准则。Order is relevant. First entries have precedence )

**4 – MinBandWidth [From] [bytes/s]**
这边有两个参数。
From 是限制来源的位置，也就是该位置受限制。他可以是完整的 hostname、网域名称或 IP。可搭配遮罩使用，例如 192.168.0.0/24 or 192.168.0.0/255.255.255.0 。另一个参数每个连线限制的最小速率，以 bytes/s 为单位，-1 代表无限制。

Examples :
BandWidth all 102400
MinBandWidth all 50000

The example above, will have a top speed of 100kb for the 1 client. If more clients come, it will be splitted accordingly but
everyone will have at least 50kb (even if you have 50 clients)
上面的例子，一个客户端最高速度为100kb,如果有多个客户端的话，它将相应的分裂，每个至少50kb,（即使有50个客户端也一样的）
BandWidth all 50000

MinBandWidth all -1

上面的例子说明每个连线有 50kb 的速度（不限制最小速度）

**5 – LargeFileLimit [Type] [Minimum Size] [bytes/s]**
顾名思义，这设定是专门用来限制大型档案的。
Type 是指副档名，可以使用 * 代表全部。也可使用 .tgz 、 .avi 等。
Minimun Size 单位是  k bytes/s，只要超过这个 Size 就被规范在这个设定的限速中。最后一个参数就是被限制的速率囉！

Example :
LargeFileLimit .avi 500 10240

This limits .avi files over (or equal to) 500kb to 10kbytes/s

**6 – BandWidthPacket [Size]**
可能您不需要去设定这个参数！预设值为 8192，适用于任何速度。这个设定必须介于 1024 至 131072。小的封包将使得速度变慢，且更耗费系统效能；相反亦是。

**7 – BandWidthError [Error]**
这个选项是用来自订个人化错误讯息的。在预设的情况下，超过最大连线时，这个 module 将会丢出  503 HTTP\_SERVICE\_UNAVAILABLE 回应。对于大部分的人来说，他们会困扰着错误讯息，不知道为什么会这样。你可以自订一个错误讯息的页面，去解释在什么情况下会发生这种问题。但有时候错误号码  503  是不适用这个地方的。所以你可以自订一个错误号码从 300 至 599。
( 有关 HTTP 错误讯息可参考下列 Reference: HTTP Protocol Error Codes)在自订错误编码时请注意，要使用尚未被定义的号码！测试的时候，我们使用错误号码 510（510尚未被定义）

And Example, with Personalized Error Page :

ErrorDocument 510 /errors/maxconexceeded.html
BandWidthError 510
注意：有时候自订错误编码可能会有问题。但在大部分的情况下，作者已修复。

**8 – MaxConnection [From] [Max]**
这个设定有两个参数。
From 是限制来源的位置，也就是该位置受限制。他可以是完整的 hostname、网域名称或 IP。可搭配遮罩使用，例如 192.168.0.0/24 or 192.168.0.0/255.255.255.0 。第二个参数是设定最大的连线数量。假如连线超过这个数量，Apache 将丢出  503 Service Temporarily Unavailable 的讯息。在设定这个参数之前，需先指定 BandWidth 值。他不需要设定的太低，您可以设定为无限制。

Example :
BandWidth all 0
MaxConnection all 20
or
BandWidth all 0
BandWidth 192.168.0.0/24 10240
MaxConnection all 20
MaxConnection 192.168.0.0/24 5

**示范区:**
这个 module 设定可安插在 virtual host 或 directory，看你要设定在 httpd.conf 或 .htaccess 皆可！不过使用 .htaccess 别忘了把 httpd.conf 里的 下的  AllowOverride  设为  ALL  。1. 限制每个连线为 10kb/s

>

```
<Virtualhost *>
```

>
>

```
BandwidthModule On
ForceBandWidthModule On
Bandwidth all 10240
MinBandwidth all -1
```

>
>

```
Servername www.haohtml.com
ServerAdmin admin@haohtml.com
```

>
>

```
</Virtualhost>
```

2. 限制每一个连线为 1000 kb/s，最小的速率为 50kb/s，且当档案超过 500 kb 即限速为 50kb/s

>

```
<Virtualhost *>
```

>
> BandwidthModule On
> ForceBandWidthModule On
> Bandwidth all 1024000
> MinBandwidth all 50000
> LargeFileLimit * 500 50000
>
> Servername www.haohtml.com
> ServerAdmin admin@haohtml.com
>
>

```
</Virtualhost>
```

3. 限制副档名为 avi & mpeg 的档案为 20 kb/s

>

```
<Virtualhost *>
```

>
> BandwidthModule On
> ForceBandWidthModule On
> LargeFileLimit .avi 1 20000
> LargeFileLimit .mpg 1 20000
>
> Servername www.haohtml.com
> ServerAdmin admin@haohtml.com
>
>

```
</Virtualhost>
```

4. 当档案(mime)为 text 格式，限制速度为 5kb/s

>

```
<Virtualhost *>
```

>
> BandwidthModule On
> AddOutputFilterByType MOD_BW text/html text/plain
> Bandwidth all 5000
>
> Servername www.haohtml.com
> ServerAdmin admin@haohtml.com
>
>

```
</Virtualhost>
```



**使用心得:**
安装及设定的方法很简单。这是我在本机上测的，限制速度为 30720 bytes/s，如果要对特定的目录进行限制的话，可以使用

>

```
<Directory />
      LargeFileLimit * 100 1024
</Directory>
```

```
指令来实现，如果不指定特定的目录，将自动继承全局设置．
```

**9 – Status Callback**

Since v0.9, the mod can display a simple status page, showing stats from the running mod. This stats show the exact information being used by the mod
to do the limiting in that second.

For this to work, you need to set a handler on any vhost. You might want to set this under an admin vhost, or set some policies to make it private.
Your call.

Example (let’s assume the vhost is for 127.0.0.1) :

>
> SetHandler modbw-handler
>

Now, you can get the status info at [http://127.0.0.1/modbw](http://127.0.0.1/modbw)
( Or download a CSV of the stats at [http://127.0.0.1/modbw?csv](http://127.0.0.1/modbw?csv) )

> The information provided is the following :
>
> id   : 0                // This is just a correlative number for each config.

 name : work.ivn.cl,all  // The vhost name, and the scope of the rule

 lock : 0                // If the memory segment is being used (0 = no)

 count: 0                // Number of users connected to this rule

 bw   : 0                // Bandwidth currently being used by the rule

 bytes: 0              // Number of bytes last sent. Only true if count>0

 hits : 0                // Number of times anyone has accesed this rule.

Simple, yet useful !

**相关文章:**

[windows平台下apache完美限制下载速度][1]

[使用apache的rewrite功能来防迅雷](http://blog.haohtml.com/archives/8196)

 [1]: http://blog.haohtml.com/archives/8187