---
title: Nginx利用多核cpu提高性能_配置参数worker_cpu_affinity
author: admin
type: post
date: 2013-12-14T03:25:34+00:00
url: /archives/14835
categories:
 - 服务器
tags:
 - nginx

---
上篇文章我们介绍了Nginx 的优化方法,这里主要对worker\_cpu\_affinity参数详细介绍一下.(官方[http://nginx.org/en/docs/ngx\_core\_module.html#worker\_cpu\_affinity][1] )
[
][2]

## 简介

Nginx默认没有开启利用多核cpu，我们可以通过增加worker_cpu_affinity配置参数来充分利用多核cpu的性能。cpu是任务处理，计算最关键的资源，cpu核越多，性能就越好。

## 规则设定

（1）cpu有多少个核，就有几位数，1代表内核开启，0代表内核关闭

（2）worker_processes最多开启8个，8个以上性能就不会再提升了，而且稳定性会变的更低，因此8个进程够用了

## 演示实例

### **两核cpu，开启两个进程**

```

worker_processes  2;
worker_cpu_affinity 01 10;

```

01表示启用了第一个cpu内核，10表示启用了第二个cpu内核


worker_cpu_affinity 01 10;表示开启了两个进程，第一个进程对应着第一个cpu内核，第二个进程对应着第二个cpu内核

### **两核cpu，开启八个进程**

```

worker_processes  8;
worker_cpu_affinity 01 10 01 10 01 10 01 10;

```

开启了8个进程，它们分别对应了开启2个内核


### **8核cpu，开启8个进程**

```

worker_processes  8;
worker_cpu_affinity 10000000 01000000 00100000 00010000 00001000 00000100 00000010 00000001;

```

00000001表示开启第一个cpu内核，00000010表示开启第二个cpu内核，依次类推


### **8核cpu，开启2个进程**

```

worker_processes  2;
worker_cpu_affinity 10101010 01010101;

```

10101010表示开启了第2,4,6,8内核，01010101表示开始了1,3,5,7内核


2个进程对应着8个内核


## 重启nginx

配置完成后，需要重启nginx服务


```
/etc/init.d/nginx -s reload
```

[1]: http://nginx.org/en/docs/ngx_core_module.html#worker_cpu_affinity
[2]: # "收起"