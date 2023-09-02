---
title: Linux 中的 Tun/Tap 介绍
author: admin
type: post
date: 2022-05-24T01:29:53+00:00
url: /archives/31687
categories:
 - 程序开发
 - 服务器
tags:
 - Linux
 - tun

---
# TUN/TAP 设备 

在计算机中TUN与TAP是操作系统内核中的虚拟网络设备。不同于硬件设备这些虚拟的网络设备全部用软件实现，但提供了与硬件设备完全相同的功能。

我们先了解一下物理设备的工作原理
![](https://blogstatic.haohtml.com/uploads/2022/07/57e54aad529225723698841910e61c46.png)

所有主机物理网卡收到的数据包时，会先将其交给内核的 Network Stack 处理，然后通过 Socket API 通知给用户态的用户程序。

Linux 中 `Tun/Tap` 驱动程序为应用程序提供了两种交互方式:

 * 虚拟网络接口和字符设备 `/dev/net/tun`。写入字符设备 `/dev/net/tun` 的数据会发送到虚拟网络接口中；
 * 发送到虚拟网络接口中的数据也会出现在该字符设备上;

我们再看下 tun 设备的工作原理
![](https://blogstatic.haohtml.com/uploads/2022/07/ece6896a211da4167687ea55edc4f2c9.png)

用户态应用往字符设备 `/dev/tunX` 写数据时，写入的数据都会出现在TUN虚拟设备上，当内核发送一个包给 TUN 虚拟设备时，通过读这个字符设备 `/dev/tunX` 同样可以拿到包的内容。

用户态应用程序写数据到 `tun/tap` 设备后进入内核态，内核态通过TCP协议复制到用户态，最后数据再次复制到内核态并通过物理网卡转发出去，期间共经历了三次用户态与内核态的复制操作，相比传统的一次复制操作来说，开销还是比较大的，因此性能会有一定的下降，这正是它的缺点。

**TAP 设备与 TUN 设备工作方式完全相同，区别在于：**

 * TUN 设备的 `/dev/tunX` 文件收发的是 IP 层数据包，只能工作在 IP 层，无法与物理网卡做 `bridge`，但是可以通过三层交换（如 `ip_forward`）与物理网卡连通。
 * TAP 设备的 `/dev/tapX` 文件收发的是 MAC 层数据包，拥有 MAC 层功能，可以与物理网卡做 `bridge`，支持 MAC 层广播

# 应用场景 

`tun/tap` 的最主要应用场景就是 `vpn`。
基实现原理就是用到隧道技术，将无法直接发送的包通先封成允许通过的合法数据包，然后经过隧道的方式传递给对方，对方收到数据包再解包成原始数据包，再继续传递下去，直到接收端收到，然后响应并原路返回。
以下图为例![](https://blogstatic.haohtml.com/uploads/2022/07/5f71f9b79d68481b7a6bacb78bc8d079-4.png)

 1. 应用进程（用户态）发起一个请求时，数据包并不是直接通过eth0网卡流出去，而是将请求数据包写入一个 `TUN` 字符设备，此时字符设备的数据会被发到虚拟网卡上（进入内核态）。根据TUN设置的特点，凡是写到这个设备的数据都可以在设备的另一端被应用程序读出的原理，应用程序客户端VPN(`Port:28001`)不断的从TUN 设备里将数据包读出来，然后再经过物理的网卡 eth0(IP1) 网卡流出，这一步就是一个普通的应用程序的客户端发起一个请求的过程。
 2. 流出的数据包通过 eth0 (IP2)被服务端VPN（`Port:38001`）接收到，然后再将收到的数据包以同样的方式写入 TUN 设备，此时进入内核态，经过 TCP/IP 协议栈，则再次将数据包经过物理网卡eth0(IP2)出去，经过交换同机或路由器直到最终到达目的主机(目的主机非本机)。
 3. 然后目的主机将响应按原来的线路返回给发起请求应用程序。

**总结**

 * 数据包在整个流程中，需要进行一些封包解包的操作, 这个操作由设备驱动完成
 * 如果数据包目的地不是VPN（`Port:38001`）当前所在主机的话，则需要向数据流向其它机器，此时务必修改IPTABLES的来源地址进行，即需要做SNAT，否则响应数据包将无法原路返回给客户端。

# 使用方法 

我们先介绍一下在 Linux 中是如何对 `TUN/TAP` 虚拟设备进行操作创建管理和删除的，但对于如何充分利用这些设备必须通过编写程序代码来实现，在后面会给出网友整理出来的演示代码。

命令行输入 `ip help` 查看 ip 命令是否支持 `tun/tap` 工具，支持的话就会显示 `tuntap` 选项：

```
# ip help
Usage: ip [ OPTIONS ] OBJECT { COMMAND | help }
       ip [ -force ] -batch filename
where  OBJECT := { link | addr | addrlabel | route | rule | neigh | ntable |
                   tunnel | tuntap | maddr | mroute | mrule | monitor | xfrm |
                   netns | l2tp | tcp_metrics | token }
```

不支持就请升级或下载最新的 `iproute2` 工具包，也可以使用类似的 `tunctl` 工具。

先查看一下 `ip tuntap` 的基本用法

```
# ip tuntap help
Usage: ip tuntap { add | del | show | list | lst | help } [ dev PHYS_DEV ]
    [ mode { tun | tap } ] [ user USER ] [ group GROUP ]
    [ one_queue ] [ pi ] [ vnet_hdr ] [ multi_queue ] [ name NAME ]

Where:    USER  := { STRING | NUMBER }
    GROUP := { STRING | NUMBER }
```

 1. 创建 `tap/tun` 设备：

```
# ip tuntap add dev tap0 mod tap # 创建 tap
# ip tuntap add dev tun0 mod tun # 创建 tun

# ifconfig -a
```

新添加的虚拟网卡默认是 `DOWN` 状态.
对于 `tun` 类型的虚拟网卡，它的MAC地址全是 0，这个是正常的

1. 激活虚拟网卡


```
# ip link set tun0 up
# ip link set tap0 up
```

对它的操作与普通网卡的命令是一样的

1. 分配IP


```
# ip addr add 10.0.0.1/24 dev tun0
```

此时 `PING 10.0.0.1` 是可以通的。

```
# ping 10.0.0.1
PING 10.0.0.1 (10.0.0.1) 56(84) bytes of data.
64 bytes from 10.0.0.1: icmp_seq=1 ttl=64 time=0.031 ms
64 bytes from 10.0.0.1: icmp_seq=2 ttl=64 time=0.036 ms
64 bytes from 10.0.0.1: icmp_seq=3 ttl=64 time=0.031 ms
64 bytes from 10.0.0.1: icmp_seq=4 ttl=64 time=0.037 ms
```

1. 删除 `tap/tun` 设备：


```
# ip tuntap del dev tap0 mod tap # 删除 tap
# ip tuntap del dev tun0 mod tun # 删除 tun
```

**代码演示**

上面我们手动创建了虚拟网卡，但没有办法测试网卡的使用效果，下面是一段实现虚拟网卡读取的演示代码，代码摘自：

```
#include <net/if.h>
#include <sys/ioctl.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
#include <sys/types.h>
#include <linux/if_tun.h>
#include<stdlib.h>
#include<stdio.h>

int tun_alloc(int flags)
{

    struct ifreq ifr;
    int fd, err;
    char *clonedev = "/dev/net/tun";

    if ((fd = open(clonedev, O_RDWR)) < 0) {
        return fd;
    }

    memset(&ifr, 0, sizeof(ifr));
    ifr.ifr_flags = flags;

    if ((err = ioctl(fd, TUNSETIFF, (void *) &ifr)) < 0) {
        close(fd);
        return err;
    }

    printf("Open tun/tap device: %s for reading...n", ifr.ifr_name);

    return fd;
}

int main()
{

    int tun_fd, nread;
    char buffer[1500];

    /* Flags: IFF_TUN   - TUN device (no Ethernet headers)
     *        IFF_TAP   - TAP device
     *        IFF_NO_PI - Do not provide packet information
     */
    tun_fd = tun_alloc(IFF_TUN | IFF_NO_PI);

    if (tun_fd < 0) {
        perror("Allocating interface");
        exit(1);
    }

    while (1) {
        nread = read(tun_fd, buffer, sizeof(buffer));
        if (nread < 0) {
            perror("Reading from interface");
            close(tun_fd);
            exit(1);
        }

        printf("Read %d bytes from tun/tap devicen", nread);
    }
    return 0;
}
```

演示

```
#--------------------------第一个shell窗口----------------------
#将上面的程序保存成tun.c，然后编译
dev@debian:~$ gcc tun.c -o tun

#启动tun程序，程序会创建一个新的tun设备，
#程序会阻塞在这里，等着数据包过来
dev@debian:~$ sudo ./tun
Open tun/tap device tun1 for reading...
Read 84 bytes from tun/tap device
Read 84 bytes from tun/tap device
Read 84 bytes from tun/tap device
Read 84 bytes from tun/tap device

#--------------------------第二个shell窗口----------------------
#启动抓包程序，抓经过tun1的包
# tcpdump -i tun1
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on tun1, link-type RAW (Raw IP), capture size 262144 bytes
19:57:13.473101 IP 192.168.3.11 > 192.168.3.12: ICMP echo request, id 24028, seq 1, length 64
19:57:14.480362 IP 192.168.3.11 > 192.168.3.12: ICMP echo request, id 24028, seq 2, length 64
19:57:15.488246 IP 192.168.3.11 > 192.168.3.12: ICMP echo request, id 24028, seq 3, length 64
19:57:16.496241 IP 192.168.3.11 > 192.168.3.12: ICMP echo request, id 24028, seq 4, length 64

#--------------------------第三个shell窗口----------------------
#./tun启动之后，通过ip link命令就会发现系统多了一个tun设备，
#在我的测试环境中，多出来的设备名称叫tun1，在你的环境中可能叫tun0
#新的设备没有ip，我们先给tun1配上IP地址
dev@debian:~$ sudo ip addr add 192.168.3.11/24 dev tun1

#默认情况下，tun1没有起来，用下面的命令将tun1启动起来
dev@debian:~$ sudo ip link set tun1 up

#尝试ping一下192.168.3.0/24网段的IP，
#根据默认路由，该数据包会走tun1设备，
#由于我们的程序中收到数据包后，啥都没干，相当于把数据包丢弃了，
#所以这里的ping根本收不到返回包，
#但在前两个窗口中可以看到这里发出去的四个icmp echo请求包，
#说明数据包正确的发送到了应用程序里面，只是应用程序没有处理该包
dev@debian:~$ ping -c 4 192.168.3.12
PING 192.168.3.12 (192.168.3.12) 56(84) bytes of data.

--- 192.168.3.12 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3023ms
```

# 参考资料 

 *
 *
 *
 *
 *
 *
 *