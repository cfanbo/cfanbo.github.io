---
title: docker容器调试利器nicolaka/netshoot
author: admin
type: post
date: 2019-07-13T05:39:35+00:00
url: /archives/19080
categories:
 - 系统架构
tags:
 - docker

---
## 背景 

在日常工作中，我们一般会将容器进行精简，将其大小压缩到最小，以此来提高容器部署效率，参考[小米云技术 – Docker最佳实践：5个方法精简镜像][1]。但有一个比较尴尬的问题就是对容器排障，由于容器里没有了我们日常工作中用到许多排障命令，如top、ps、netstat等，所以想排除故障的话，常用的做法是安装对应的命令，如果容器过多的话，再这样搞就有些麻烦了，特别是在一些安装包源速度很慢的情况。

## 解决方案 

今天发现一篇文章（[简化 Pod 故障诊断：kubectl-debug 介绍][2]）介绍针对此类问题的解决方案的，这里介绍的是一个叫做 `kubectl-debug` 的命令，主要由国内知名的PingCAP公司出品的，主要是用在k8s环境中的。我们知道容器里主要两大技术，一个是用**cgroup来实现容器资源的限制**，一个是用**Namespace来实现容器的资源隔离的**）。（kubectl-debug 命令是基于一个工具包（） 来实现的，其原理是利用将一个工具包容器添加到目标容器所在的Pod里，实现和目标容器的Network Namespace一致，从而达到对新旧容器进程的相互可见性，这样我们就可以直接在目标容器里操作这些命令。所以在平时开发环境中，可以很方便的利用此原理直接使用这个工具包来实现对容器的排障。

> 在 Kubernetes 项目中，这些容器则会被划分为一个“Pod”，Pod 里的容器共享同一个 Network Namespace、同一组数据卷，从而达到高效率交换信息的目的。这些容器应用就可以通过 Localhost 通信，通过本地磁盘目录交换文件。

netshoot包含一组强大的工具，如图所示![](https://blog.haohtml.com/wp-content/uploads/2019/07/netshoot-1024x717.png)

工具包清单

```
apache2-utils
bash
bind-tools
bird
bridge-utils
busybox-extras
calicoctl
conntrack-tools
ctop
curl
dhcping
drill
ethtool
file
fping
iftop
iperf
iproute2
iptables
iptraf-ng
iputils
ipvsadm
libc6-compat
liboping
mtr
net-snmp-tools
netcat-openbsd
netgen
nftables
ngrep
nmap
nmap-nping
openssl
py-crypto
py2-virtualenv
python2
scapy
socat
strace
tcpdump
tcptraceroute
util-linux
vim
```

看了上图不得不说几乎包含了所有的调度命令。

使用也很方便，只需要一条命令即可

```
$ docker run -it --net container: nicolaka/netshoot
```

参数 `--net` 是 `--network` 的缩写，有四种值，用法参考： [https://docs.docker.com/engine/reference/run/#network-settings](https://docs.docker.com/engine/reference/run/#network-settings)

执行命令后，会自动创建一个镜像为 nicolaka/netshoot 的容器，

Docker 提供的这个 `--net` 参数，可以让你启动一个容器并“加入”到另一个容器的 Network Namespace 里，并共享一个网络栈，即Namespace 和目标容器的一样，这样就实现了合并命令工具到**目标容器** 里。如果执行命令 `hostname` 命令的话，结果显示的是目标 “**容器ID**” 。此时就可以在容器里执行常用的一些ps、 top、netstat、iftop 之类的命令。

```
➜  ~ docker run -it --rm --network container:mysql80 nicolaka/netshoot
                    dP            dP                           dP
                    88            88                           88
88d888b. .d8888b. d8888P .d8888b. 88d888b. .d8888b. .d8888b. d8888P
88'  `88 88ooood8   88   Y8ooooo. 88'  `88 88'  `88 88'  `88   88
88    88 88.  ...   88         88 88    88 88.  .88 88.  .88   88
dP    dP `88888P'   dP   `88888P' dP    dP `88888P' `88888P'   dP

Welcome to Netshoot! (github.com/nicolaka/netshoot)
root @ /
 [1] 🐳  → ls
bin    dev    etc    home   lib    lib64  media  mnt    opt    proc   root   run    sbin   srv    sys    tmp    usr    var

root @ /
 [2] 🐳  → hostname
7ff422d3f75d

root @ /
 [3] 🐳  → ps
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/bash -l
   15 root      0:00 ps

root @ /
 [4] 🐳  → netstat -an | grep LISTEN
tcp        0      0 :::33060                :::*                    LISTEN
tcp        0      0 :::3306                 :::*                    LISTEN
unix  2      [ ACC ]     STREAM     LISTENING      18007 /var/run/mysqld/mysqld.sock
unix  2      [ ACC ]     STREAM     LISTENING      19694 /var/run/mysqld/mysqlx.sock

root @ /
 [5] 🐳  →
```

上面我们添加了`--rm` 参数，主要是为了使用完毕后，及时清除临时容器相关的资源。这里我们将临时容器的Namespace和mysql80 容器相同

mysql80容器的ID为 `7ff422d3f75d`，即hostname 命令的输出结果。

现在我们先不要从这个容器里退出，再开启一个新终端进入到临时容器(`bb47226c955e`)里

```
➜  ~ docker exec -it bb4 /bin/bash
bash-5.0# hostname
7ff422d3f75d
bash-5.0# ps
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/bash -l
   22 root      0:00 /bin/bash
   28 root      0:00 ps
bash-5.0# netstat -an | grep LISTEN
tcp        0      0 :::33060                :::*                    LISTEN
tcp        0      0 :::3306                 :::*                    LISTEN
unix  2      [ ACC ]     STREAM     LISTENING      18007 /var/run/mysqld/mysqld.sock
unix  2      [ ACC ]     STREAM     LISTENING      19694 /var/run/mysqld/mysqlx.sock
bash-5.0#

```

从hostname和进程ID的结果来看，进入的还是msyql80容器。

最后执行 exit 或者 logout 命令退出容器，此时临时容器及其volume将自动被删除(参数–rm)。

## 总结 

这里主要考察了容器的Namespace隔离性的原理，通过将一个容器所有进程加入到目标容器的Namespace，从而实现了两个容器进程的相互可见。

## 问题延伸 

这里抛出另一个问题，如果两个容器都存在一个一模一样的进程，这个时候会出错吗？如果不会的话，为什么(应用存储路径不一样？或者进程PID不一样)？我们可见的是哪个容器的里程，临时容器还是目标容器呢？

把一个进程分配到一个指定的Namespace下的原理请参考 https://time.geekbang.org/column/article/18119 

 [1]: https://mp.weixin.qq.com/s/S1Ib08SpQbf1SCbCutUoqQ
 [2]: http://www.dockone.io/article/9032