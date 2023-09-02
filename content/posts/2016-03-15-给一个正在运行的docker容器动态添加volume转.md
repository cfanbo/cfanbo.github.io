---
title: 给一个正在运行的Docker容器动态添加Volume(转)
author: admin
type: post
date: 2016-03-15T06:27:56+00:00
url: /archives/16799
categories:
 - 系统架构
tags:
 - docker

---
之前有人问我Docker容器启动之后还能否再挂载卷，考虑mnt命名空间的工作原理，我一开始认为这很难实现。不过现在我认为是它实现的。

简单来说，要想将磁盘卷挂载到正在运行的容器上，我们需要：

 * 使用[nsenter][1]将包含这个磁盘卷的整个文件系统mount到临时挂载点上；
 * 从我们想当作磁盘卷使用的特定文件夹中创建绑定挂载（bind mount）到这个磁盘卷的位置；
 * umount第一步创建的临时挂载点。

#### **注意事项**

在下面的示例中，我故意包含了$符号来表示这是Shell命令行提示符，以帮助大家区分哪些是你需要输入的，哪些是机器回复的。有一些多行命令，我也继续用>。我知道这样使得例子里的命令无法轻易得被拷贝粘贴。如果你想要拷贝粘贴代码，请查看文章最后的示例脚本。

#### **详细步骤**

下面示例的前提是你已经使用如下命令启动了一个简单的名为charlie的容器：

```
$ docker run --name charlie -ti ubuntu bash

```

我们需要做的是将宿主文件夹 `/home/jpetazzo/Work/DOCKER/docker` 挂载到容器里的 `/src` 目录。好了，让我们开始吧。

**nsenter**

首先，我们需要[nsenter][1]以及docker-enter帮助脚本。为什么？因为我们要从容器中mount文件系统。由于安全性的考虑，容器不允许我们这么做。使用nsenter，我们可以突破上述安全限制，在容器的上下文（严格地说，是命名空间）中运行任意命令。当然，这必须要求拥有Docker宿主机的root权限。
[nsenter][1]最简单的安装方式是和 `docker-enter` 脚本关联执行：

```
$ docker run --rm -v /usr/local/bin:/target jpetazzo/nsenter

```

更多细节，请查看[nsenter][1]项目主页。

**找到文件系统**

我们想要在容器里挂载包含宿主文件夹（ `/home/jpetazzo/Work/DOCKER/docker`）的文件系统。那我们就需要找出哪个文件系统包含这个目录。

首先，我们需要canonicalize（或者解除引用）文件，以防这是一个符号链接，或者它的路径包含符号链接：

```
$ readlink --canonicalize /home/jpetazzo/Work/DOCKER/docker
/home/jpetazzo/go/src/github.com/docker/docker

```

哈，这的确是一个符号链接！让我们将其放入一个环境变量中：

```
$ HOSTPATH=/home/jpetazzo/Work/DOCKER/docker
$ REALPATH=$(readlink --canonicalize $HOSTPATH)

```

接下来，我们需要找出哪个文件系统包含这个路径。我们使用一个有点让人意想不到的工具来做，它就是 `df`：

```
$ df $REALPATH
Filesystem     1K-blocks      Used Available Use% Mounted on
/sda2          245115308 156692700  86157700  65% /home/jpetazzo

```

使用-P参数（强制使用POSIX格式，以防是exotic df，或者是其他人在Solaris或者BSD系统上装Docker时运行的df），将结果也放到一个变量里：

```
$ FILESYS=$(df -P $REALPATH | tail -n 1 | awk '{print $6}')

```

**找到文件系统的设备（和sub-root）**

现在，系统里已经没有绑定挂载（bind mounts）和BTRFS子卷了，我们仅仅需要查看/proc/mounts，找到对应于 `/home/jpetazzo` 文件系统的设备就可以了。但是在我的系统里， `/home/jpetazzo` 是BTRFS池的子卷，要想得到子卷的信息（或者bind mount信息），需要查看 `/proc/self/moutinfo`。

如果你从来没有听说过mountinfo，可以查看内核文档的[proc.txt][2]。

首先，得到文件系统设备信息：

```
$ while read DEV MOUNT JUNK
> do [ $MOUNT = $FILESYS ] && break
> done </proc/mounts
$ echo $DEV
/dev/sda2

```

接下来，得到sub-root信息（比如，已挂载文件系统的路径）：

```
$ while read A B C SUBROOT MOUNT JUNK
> do [ $MOUNT = $FILESYS ] && break
> done < /proc/self/mountinfo
$ echo $SUBROOT
/jpetazzo

```

很好。现在我们知道需要挂载 `/dev/sda2`。在文件系统内部，进入 `/jpetazzo`，从这里可以得到到所需文件的剩余路径（示例中是 `/go/src/github.com/docker/docker`）。
让我们计算出剩余路径：

```
$ SUBPATH=$(echo $REALPATH | sed s,^$FILESYS,,)

```



> 注意：这个方法只适用于路径里没有符号“,”的。如果你的路径里有“,”并且想使用本文方法挂载目录，请告诉我。（我需要调用Shell Triad来解决这个问题：[jessie][3]，[soulshake][4]，[tianon][5]?）

在进入容器之前最后需要做的是找到这个块设备的主和次设备号。可以使用stat：

```
$ stat --format "%t %T" $DEV
8 2

```

注意这两个数字是十六进制的，我们之后需要的是二进制。可以这么转换：

```
$ DEVDEC=$(printf "%d %d" $(stat --format "0x%t 0x%T" $DEV))

```



### 总结

还有最后一步。因为某些我无法解释的原因，一些文件系统（包括BTRFS）在挂载多次之后会更新/proc/mounts里面的设备字段。也就是说，如果我们在容器里创建了名为/tmpblkdev的临时块设备，并用其挂载我们自己的文件系统，那么文件系统（在宿主机器里！）会显示为 `/tmpblkdev`，而不是 `/dev/sda2`。这听起来无所谓，但实际上这会让之后试图得到文件系统块设备的操作都失败。

长话短说，我们想要确保块设备节点在容器里位于和宿主机器上的同一个路径下。

需要这么做：

```
$ docker-enter charlie -- sh -c \
> "[ -b $DEV ] || mknod --mode 0600 $DEV b $DEVDEC"

```

创建临时挂载点挂载文件系统：

```
$ docker-enter charlie -- mkdir /tmpmnt
$ docker-enter charlie -- mount $DEV /tmpmnt

```

确保卷挂载点存在，bind mount卷：

```
$ docker-enter charlie -- mkdir -p /src
$ docker-enter charlie -- mount -o bind /tmpmnt/$SUBROOT/$SUBPATH /src

```

删除临时挂载点：

```
$ docker-enter charlie -- umount /tmpmnt
$ docker-enter charlie -- rmdir /tmpmnt

```

(我们并不清除设备节点。一开始就检查设备是否存在可能有点多余，但是现在再检查就已经很复杂了。)

大功告成！

#### 让一切自动化

下面这段可以直接拷贝粘贴了。

```
#!/bin/sh
set -e
CONTAINER=charlie
HOSTPATH=/home/jpetazzo/Work/DOCKER/docker
CONTPATH=/src

REALPATH=$(readlink --canonicalize $HOSTPATH)
FILESYS=$(df -P $REALPATH | tail -n 1 | awk '{print $6}')

while read DEV MOUNT JUNK
do [ $MOUNT = $FILESYS ] && break
done  </proc/mounts
[ $MOUNT = $FILESYS ] # Sanity check!

\while read A B C SUBROOT MOUNT JUNK
\do [ $MOUNT = $FILESYS ] && break
\done < /proc/self/mountinfo
[ $MOUNT = $FILESYS ] # Moar sanity check!

SUBPATH=$(echo $REALPATH | sed s,^$FILESYS,,)
DEVDEC=$(printf "%d %d" $(stat --format "0x%t 0x%T" $DEV))

docker-enter $CONTAINER -- sh -c \
     "[ -b $DEV ] || mknod --mode 0600 $DEV b $DEVDEC"
docker-enter $CONTAINER -- mkdir /tmpmnt
docker-enter $CONTAINER -- mount $DEV /tmpmnt
docker-enter $CONTAINER -- mkdir -p $CONTPATH
docker-enter $CONTAINER -- mount -o bind /tmpmnt/$SUBROOT/$SUBPATH $CONTPATH
docker-enter $CONTAINER -- umount /tmpmnt
docker-enter $CONTAINER -- rmdir /tmpmnt

```



#### 状态和限制

上述方法不适用于不基于块设备的文件系统，只有在 `/proc/mounts` 能正确得到块设备节点（上面谈到，并不总是能正确得到）的时候才能起作用。另外，我只测试了我自己的环境，没有在云实例之类的环境里测试过，但是我很想知道在那里是否适用。

**原文链接：[Attach a volume to a container while it is running][6] （翻译：崔婧雯 校对：李颖杰）**

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
译者介绍
崔婧雯，现就职于VMware，高级软件工程师，负责桌面虚拟化产品的质量保证工作。曾在IBM WebSphere业务流程管理软件担任多年系统测试工作。对虚拟化，中间件技术有浓厚的兴趣。

http://dockone.io/article/149

 [1]: https://github.com/jpetazzo/nsenter
 [2]: https://www.kernel.org/doc/Documentation/filesystems/proc.txt
 [3]: https://twitter.com/frazelledazzell
 [4]: https://twitter.com/s0ulshake
 [5]: https://twitter.com/tianon
 [6]: http://jpetazzo.github.io/2015/01/13/docker-mount-dynamic-volumes/