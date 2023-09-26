---
title: 'Linux下两种 DNAT 用法的差异'
author: admin
type: post
date: 2022-10-30T12:15:13+00:00
url: /archives/31967
toc: true
categories:
 - 程序开发
tags:
 - iptables

---
前段时间使用 `iptables` 的 `DNAT` 实现一个业务需求的时候，遇到了一些问题这里将其整个过程记录下来。

## 需求 

这里假设开发机地址为 `192.168.3.80`，要实现的需求是当用户在开发机访问一个IP地址 `192.168.3.196`时，将请求转发到另一台机器 `192.168.3.58`，很明显直接使用 `DNAT` 来实现即可。

## 问题现象 

iptables 命令如下

```
sudo iptables -t nat -F
sudo iptables -t nat -A PREROUTING -d 192.168.3.196 -p tcp --dport 8080 -j DNAT --to-destination 192.168.3.58:8080
sudo iptables -t nat -A POSTROUTING -d 192.168.3.58 -p tcp --dport 8080 -j SNAT --to-source 192.168.3.196:8080
```

这时在开发机器访问

```
curl http://192.168.3.196:8080
```

发现提示错误

```
curl: (7) Failed to connect to 192.168.3.196 port 8080: Connection refused
```

奇怪了，竟然不能访问，确认路由规则是写入成功的。网上查找了一些资料好像全是这种写法，只不过用法有怕差异，这时直觉告诉我应该对 DNAT 理解不到位，遗漏了一些重要的知识点。

上面这种写法一般都是将开发机当作一个`中转服务器`或`跳板`来使用，多种情况下都有一个公网ip，与我们的真正需求有一些不一样。

现在我们再以将其视为中转服务器的角色测试一次，当然这个规则不能直接使用上面的这个，需要把访问的目标ip更换成开发机器的IP地址。

```
sudo iptables -t nat -F
sudo iptables -t nat -A PREROUTING -d 192.168.3.80 -p tcp --dport 8080 -j DNAT --to-destination 192.168.3.58:8080
sudo iptables -t nat -A POSTROUTING -d 192.168.3.58 -p tcp --dport 8080 -j SNAT --to-source 192.168.3.80:8080
```

第一条是数据包出去规则，需要做DNAT，第二条是数据包回来规则，必须做一次 SNAT，否则数据包将去无回，无法响应。

这时再找一台机器访问 `curl 192.168.3.80:8080`，可以看到响应结果符合预期。

## 问题分析 

现在我们基本确认了是我们的用法不对，到底是哪里出错了呢？这里我们一起看一下这张 `iptables` 数据流向图。![](https://blogstatic.haohtml.com/uploads/2021/10/d2b5ca33bd970f64a6301fa75ae2eb22.png?x-oss-process=image%2Fformat,webp)iptables Processing Flowchart

从图中可以看到，对于数据流入一共有两类，一类是外部数据包流入 ,即左侧的 `Incoming Packet`；另一类是本机生成的数据包流入，即右侧的 `Locally generated Packet`，对于对数据包的流出只有一处，即下方的 `Outgoing Packet`。

对于数据包首个经过的表是不一样的，对于外部流入的数据包首个经过的是`PREROUTING` 表，而对于本地生成的数据包而言经过的是 `OUTPUT` 这个表，最后统一从同一个地方流出。

也就是说针对不的类型的包，经过的表是不同的，这个正是我们最上面失败的原因。

## 解决问题 

我们要实现的场景其实是 `Locally generated Packet` 这类，所以使用的表应该是 `OUTPUT`才是正确的，现在我们清除原来的规则，重新写入新规则测试一下

```
sudo iptables -t nat -F
sudo iptables -t nat -A OUTPUT -d 192.168.3.196 -p tcp --dport 8080 -j DNAT --to-destination 192.168.3.58:8080
```

注意对于 `SNAT`而言，只对 `INPUT/POSTROUTING`有效。

再次测试 `curl 192.168.3.196:8080` 响应正常。

## 总结 

针对 iptables 的 `DNAT` 的实现，需要根据数据包的来源不同而采用不同的处理方法，一共分外部数据包和本地数据包两类。其中对于外部数据包除了做 DNAT外，还要再做一个 SNAT 规则，否则数据包将有去无回；而对于本地数据包而言，只需要在 OUTPUT 表中做一个 DNAT 即可，并不需要SNAT，同时也不支持 SNAT。对于SNAT 只对 `INPUT/POSTROUTEING` 才有效。