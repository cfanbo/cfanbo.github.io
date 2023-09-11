---
title: docker exec 命令原理
author: admin
type: post
date: 2019-07-26T11:59:17+00:00
url: /archives/19151
categories:
 - 系统架构
tags:
 - docker

---
我们经常使用 docker exec 命令进入到一个容器里进行一些操作，那么这个命令是如果进入到容器里的呢？想必大家都知道用到了Namespace来实现，但至于底层实现原理是什么，想必都不是特别清楚吧。

我们知道容器的本质其实就是一个进程，每个进程都有一个Pid，至于容器的Pid值可以通过 docker inspect container_id 来查看，我们这里是一个Python应用容器，我们看一下他的 Pid值

```
docker inspect --format '{{ .State.Pid }}'  4ddf4638572d
25686
```

而每个进程都有自己的Namespace，你可以通过查看宿主机的 proc 文件，看到这个 25686 进程的所有 Namespace 对应的文件

```
ls -l  /proc/25686/ns
total 0
lrwxrwxrwx 1 root root 0 Aug 13 14:05 cgroup -> cgroup:[4026531835]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 ipc -> ipc:[4026532278]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 mnt -> mnt:[4026532276]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 net -> net:[4026532281]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 pid -> pid:[4026532279]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 pid_for_children -> pid:[4026532279]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 user -> user:[4026531837]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 uts -> uts:[4026532277]

```

可以看到，一个进程的每种 Linux Namespace，都在它对应的 /proc/[进程号]/ns 下有一个对应的虚拟文件，并且链接到一个真实的 Namespace 文件上。可以看到这个容器对应的**Net Namespace** Id为 4026532281

进入容器的命令为

```
$ docker exec -it 4ddf4638572d /bin/sh
```

对于我们执行的 `/bin/sh` 命令，进程PID在宿主机器上是可以看到的。

# 在宿主机上 

```
$ ps aux | grep /bin/bash
 root     28499  0.0  0.0 19944  3612 pts/0    S    14:15   0:00 /bin/bash
```

实际上，Linux Namespace 创建的隔离空间虽然看不见摸不着，但一个进程的 Namespace 信息在宿主机上是确确实实存在的，并且是以一个文件的方式存在。

我们现在看一下这两个进程对应的Namespace信息

```
ls -l /proc/28499/ns/net
lrwxrwxrwx 1 root root 0 Aug 13 14:18 /proc/28499/ns/net -> net:[4026532281]

$ ls -l  /proc/25686/ns/net
lrwxrwxrwx 1 root root 0 Aug 13 14:05 /proc/25686/ns/net -> net:[4026532281]
```

在 `/proc/[PID]/ns/net` 目录下，这个 PID=28499 进程，与我们前面的 Docker 容器进程（PID=25686）指向的 Network Namespace 文件完全一样。这说明这两个进程，共享了这个名叫 net:[4026532281] 的 Network Namespace。

此外，Docker 还专门提供了一个参数，可以让你启动一个容器并“加入”到另一个容器的 Network Namespace 里，这个参数就是 -net，比如:

```
docker run -it --net container:4ddf4638572d busybox ifconfig
```

这样，我们新启动的这个容器，就会直接加入到 ID=4ddf4638572d 的容器，也就是我们前面的创建的 Python 应用容器（PID=25686）的 Network Namespace 中。参考文章：[docker容器调试利器nicolaka/netshoot][1]

而如果我指定–net=host，就意味着这个容器不会为进程启用 Network Namespace。这就意味着，这个容器拆除了 Network Namespace 的“隔离墙”，所以，它会和宿主机上的其他普通进程一样，直接共享宿主机的网络栈。这就为容器直接操作和使用宿主机网络提供了一个渠道。

现在我们知道了 docker exec 的原理，那么对于 docker run -v /home:/test … 是如何实际目录挂载的呢？

建议参考 https://time.geekbang.org/column/article/18119

 [1]: https://blog.haohtml.com/archives/19080