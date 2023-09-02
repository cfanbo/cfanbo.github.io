---
title: Memcache mutex设计模式
author: admin
type: post
date: 2010-11-24T05:16:12+00:00
url: /archives/6772
IM_contentdowned:
 - 1
categories:
 - 系统架构
tags:
 - memcache

---
周六的S2 [Web 2.0技术沙龙][1]上介绍了memcache中使用mutex场景(文后要演讲稿)，有网友对详情感兴趣，简单介绍如下。

### 场景

Mutex主要用于有大量并发访问并存在cache过期的场合，如

 * 首页top 10, 由数据库加载到memcache缓存n分钟
 * 微博中名人的content cache, 一旦不存在会大量请求不能命中并加载数据库
 * 需要执行多个IO操作生成的数据存在cache中, 比如查询db多次

### 问题

在大并发的场合，当cache失效时，大量并发同时取不到cache，会同一瞬间去访问db并回设cache，可能会给系统带来潜在的超负荷风险。**我们曾经在线上系统出现过类似故障**。

### 解决方法

**方法一**
在load db之前先add一个mutex key, mutex key add成功之后再去做加载db, 如果add失败则sleep之后重试读取原cache数据。为了防止死锁，mutex key也需要设置过期时间。伪代码如下
(_注：下文伪代码仅供了解思路，可能存在bug，欢迎随时指出。_)

```
if (memcache.get(key) == null) {
    // 3 min timeout to avoid mutex holder crash
    if (memcache.add(key_mutex, 3 * 60 * 1000) == true) {
        value = db.get(key);
        memcache.set(key, value);
        memcache.delete(key_mutex);
    } else {
        sleep(50);
        retry();
    }
}
```

**方法二**
在value内部设置1个超时值(timeout1), timeout1比实际的memcache timeout(timeout2)小。当从cache读取到timeout1发现它已经过期时候，马上延长timeout1并重新设置到cache。然后再从数据库加载数据并设置到cache中。伪代码如下

```
v = memcache.get(key);
if (v == null) {
    if (memcache.add(key_mutex, 3 * 60 * 1000) == true) {
        value = db.get(key);
        memcache.set(key, value);
        memcache.delete(key_mutex);
    } else {
        sleep(50);
        retry();
    }
} else {
    if (v.timeout <= now()) {
        if (memcache.add(key_mutex, 3 * 60 * 1000) == true) {
            // extend the timeout for other threads
            v.timeout += 3 * 60 * 1000;
            memcache.set(key, v, KEY_TIMEOUT * 2);

            // load the latest value from db
            v = db.get(key);
            v.timeout = KEY_TIMEOUT;
            memcache.set(key, value, KEY_TIMEOUT * 2);
            memcache.delete(key_mutex);
        } else {
            sleep(50);
            retry();
        }
    }
}
```

相对于方案一
优点：避免cache失效时刻大量请求获取不到mutex并进行sleep
缺点：代码复杂性增大，因此一般场合用方案一也已经足够。

方案二在Memcached FAQ中也有详细介绍 [How to prevent clobbering updates, stampeding requests][2]，并且Brad还介绍了用他另外一个得意的工具 Gearman 来实现单实例设置cache的方法，见 [Cache miss stampedes][3]，不过用Gearman来解决就感觉就有点奇技淫巧了。

附：本次Web2.0技术沙龙演讲主题：微博Cache设计谈，需下载请点击演讲稿下menu/download (需登录slideshare)。

**[微博cache设计谈](http://www.slideshare.net/iso1600/cache-4842490 "微博cache设计谈")**

View more [presentations](http://www.slideshare.net/) from [Tim Y](http://www.slideshare.net/iso1600).

 [1]: http://www.s2forum.org/1/topic/
 [2]: http://code.google.com/p/memcached/wiki/FAQ#How_to_prevent_clobbering_updates,_stampeding_requests
 [3]: http://lists.danga.com/pipermail/memcached/2007-July/004858.html