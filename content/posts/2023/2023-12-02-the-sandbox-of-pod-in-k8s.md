---
title: pod sandbox 创建netns源码分析
author: admin
type: post
toc: true
date: 2023-12-02T10:15:53+00:00
url: /posts/the-sandbox-of-pod-in-k8s
keywords: sandbox、k8s、pod、containerd
categories:
- 程序开发
tags:
- k8s
- containerd
---

在上一篇《[创建Pod源码解析](https://blog.haohtml.com/archives/33163/)》文中，我们大概介绍了Pod的整体创建过程。其中有一步很重要，就是在创建三类容器之前必须先创建一个 ` sandbox` （[源码](https://github.com/kubernetes/kubernetes/blob/v1.27.3/pkg/kubelet/kuberuntime/kuberuntime_manager.go#L1079))，本篇就来分析一下sandbox这一块的 `netns` 实现过程。

对 `sandbox`  的创建由 `kubelet` 组件通过调用  [CRI](https://kubernetes.io/zh-cn/docs/concepts/architecture/cri/) 容器运行时服务来实现的，对于容器运行的实现目前市面上有多个，如 [Docker Engine](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/#docker)(不推荐)、 [containerd](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/#containerd)、[CRI-O](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/#cri-o) 等，由于目前生产环境中选择 containerd 的占大多数，所以这里我们以 `containerd` 为例来看一下其实现过程。

https://github.com/containerd/containerd/blob/32bf805e5703bc91387d047fa76625e915ac2b80/pkg/cri/server/sandbox_run.go

对 sandbox 的创建是由 cri 服务调用  `RunPodSandbox()`方法来实现的。 

```go
// RunPodSandbox creates and starts a pod-level sandbox. Runtimes should ensure
// the sandbox is in ready state.
func (c *criService) RunPodSandbox(ctx context.Context, r *runtime.RunPodSandboxRequest) (_ *runtime.RunPodSandboxResponse, retErr error) {

  ...
  
  // Save sandbox name
	sandboxInfo.AddLabel("name", name)

  // 初始化 sandbox, 注意此时的 network namespace 未指定
	// Create initial internal sandbox object.
	sandbox := sandboxstore.NewSandbox(
		sandboxstore.Metadata{
			ID:             id,
			Name:           name,
			Config:         config,
			RuntimeHandler: r.GetRuntimeHandler(),
		},
		sandboxstore.Status{
			State: sandboxstore.StateUnknown,
		},
	)

	if _, err := c.client.SandboxStore().Create(ctx, sandboxInfo); err != nil {
		return nil, fmt.Errorf("failed to save sandbox metadata: %w", err)
	}
  
  ...
  
  // Setup the network namespace if host networking wasn't requested.
	if !hostNetwork(config) && !userNsEnabled {
		netStart := time.Now()

		var netnsMountDir = "/var/run/netns"
		if c.config.NetNSMountsUnderStateDir {
			netnsMountDir = filepath.Join(c.config.StateDir, "netns")
		}
    
    // 创建 network namespace! 本文重点关注函数
		sandbox.NetNS, err = netns.NewNetNS(netnsMountDir)
    
    // 指定 network namespace 路径
    // Update network namespace in the store, which is used to generate the container's spec
		sandbox.NetNSPath = sandbox.NetNS.GetPath()
    
    ...

		if err := c.setupPodNetwork(ctx, &sandbox); err != nil {
			return nil, fmt.Errorf("failed to setup network for sandbox %q: %w", id, err)
		}

  }  
}
```

这里我们了解一个上面提到的  `/var/run/netns` 这个特殊的目录，以下来自GPT的解释：

在 Linux 中，`/var/run/netns` 目录的作用是存储网络命名空间的符号链接。网络命名空间是一种 Linux 内核提供的功能，用于将网络资源进行隔离，使得每个网络命名空间可以有独立的网络栈、网络接口、路由表和防火墙规则等。

在 `/var/run/netns` 目录中，每个网络命名空间都会有一个对应的符号链接。符号链接的名称通常是命名空间的名称，指向实际的网络命名空间文件或目录，它们的路径在 `/proc` 文件系统下的 `/proc/[PID]/ns/net`。这些符号链接可以用于标识和引用特定的网络命名空间。

通过在 `/var/run/netns` 目录下创建和管理符号链接，用户可以方便地引用和操作不同的网络命名空间。例如，可以使用工具如 `ip netns` 来创建、删除和管理网络命名空间，以及在不同的网络命名空间之间设置网络接口、路由和防火墙规则等操作。

总之，`/var/run/netns` 目录提供了一个简便的方式来识别和引用 Linux 系统中的网络命名空间。

上面的解释提到了两点：一个是` /var/run/netns` 目录里存放的是命全名空间符号链接，另一个是实际的命名空间对应的路径目录为 `/proc/[PID]/ns/net`。也就是说将 /proc/[pid]/ns/net 实际全名空间挂载到 /var/run/netns 目录里的链接文件，两个目录里的文件存在一定的对应关系。

另外，在 Linux 中，每个进程都有一个默认的网络命名空间。这意味着每个进程都具有自己独立的网络栈、网络接口、路由表和防火墙规则，从而实现了网络隔离。默认情况下，多个进程不会共享同一个网络命名空间。

如果将多个进程置于同一个network namespace，则这些进程将共享相同的网络资源和配置。



# netns 操作

这里我们介绍一下在Linux中通过 ip netns 命令来实现将进程添加到网络命名空间的过程。步骤：

1. 在终端创建一个网络命名空间文件 mynet。将此终端视为 Admin 终端，主要用来观察netns 变更
2. 新开终端，创建一个网络命名空间，将当前 network namespace 文件挂载到上面创建的 mynet 文件
3. 在Admin 查看 mynet 下包含的network namespace, 其对应的是进程PID信息（容器的本质是一个进程，因此这里的pid可以视为一个容器）

## 1. 创建一个mynet 文件

打开一个终端（Admin终端），这里我们用 `ip netns` 创建一个命令空间

```shell
$ sudo ip netns add mynet
```

你也可以这个网络命名空间设置一个ID，如

```shell
$ sudo ip netns set mynet 12345
```

需要注意，这个标识一经指定，后期将无法修改。

查看当前所有的网络命名空间

```shell
$ sudo ip netns list
mynet (id: 12345)
```

这里显示了network namespace ID

或

```shell
$ ls -al /var/run/netns/
total 0
drwxr-xr-x  2 root root   60 Dec  2 09:17 .
drwxr-xr-x 33 root root 1000 Dec  2 09:17 ..
-r--r--r--  1 root root    0 Dec  2 09:17 mynet
```

可以看到在这个目录里出现了大小为 `0` 的空文件。

查看 network namespace 中所有进程PID

```shell
$ sudo ip netns pids mynet
```

由于刚刚创建，这里是没有任何输出的。

如果想查看指定进程所属netns

```shell
$ sudo ip netns identify [PID]
```

查看mynet 信息

```shell
$ stat /var/run/netns/mynet
  File: /var/run/netns/mynet
  Size: 0         	Blocks: 0          IO Block: 4096   regular empty file
Device: 4h/4d	Inode: 4026532942  Links: 1
Access: (0444/-r--r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2023-12-02 11:47:47.632117163 +0800
Modify: 2023-12-02 11:47:47.632117163 +0800
Change: 2023-12-02 11:47:47.632117163 +0800
```

这里的mynet文件的 Inode 为 `4026532942`,  它代表了当前文件network namespace ID，

查看当前默认网络命名空间

```shell
$ readlink /proc/$$/ns/net
net:[4026531992]
```

不同的网络命名空间是不一样的，当前netns ID为  `4026531992`，它与上面我们创建的mynet 是没有任何关系 的，每个会话（准备的说是进程， 这里一般指 bash）都有属于自己的网络命名空间。

## 2. 新开终端，创建一个命名空间

新开一个终端 A，查看当前默认网络命名空间

```shell
$ readlink /proc/$$/ns/net
net:[4026531992]
```

这里是和前面的终端是一样的，个人理解的是同一个bash原因。

通过 unshare 命令创建新的 network namespace，并在新的 namespace 中启动新的 bash：

```shell
$ sudo unshare --net bash
```

注意这时进入了一个全新的namespace，因此网络命名空间是不一样的。

```shell
$ readlink /proc/$$/ns/net
net:[4026532828]
```



下面重点来了，需要将当前网络命名空间加入到 mynet 里面。

通过绑定挂载把当前 bash 进程的 network namespace 文件挂载到前面创建的 mynet 文件

```shell
$ mount --bind /proc/$$/ns/net /var/run/netns/mynet
$ ls -i /var/run/netns/mynet
```

## 3. 验证

这时我们再回到 Admin 终端，再次查看 mynet 网络全名空间里的所有进程pid信息

```shell
$ sudo ip netns pids mynet
9615
```

可以看到此命名空间有了一个pid为 `9615`的进程，而这个进程正是我们上面 `unshare --net` 命令中的 bash 进程PID, 查找这个对应的进程信息

```shell
$ ps aux | grep 9615 | grep -v grep
root        9615  0.0  0.1   7428  4196 pts/1    S    11:16   0:00 bash
```

至此，我们将一个进程加入到了一个 mynet 的网络命名空间中。这时在mynet 这个network namespace中只有bash 一个进程，因此在 终端A 执行 `netstat -lanp` 是没有任何输出的。

```shell
$ netstat -lntp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
```

## 4. 同命名空间相互进程可见

我们还可以将其它进程用同样的办法加入进来，这样同属于mynet这个network namespace里的进程之间就算于在同一个netns，它们之间可以通过 127.0.0.1 进行一些网络通讯。

再新开一个终端B，我们用python启用一个 webserver，这里我们使用另一个命令 `nsenter`

```shell
$ sudo nsenter --net=/var/run/netns/mynet python3 -m http.server
```

这里回到 Admin 终端，查看mynet中的进程信息

```shell
$ sudo ip netns pids mynet
9615
32337
```

同时在终端A中，再次执行 netstat 命令

```shell
$ netstat -lntp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp6       0      0 :::8000                 :::*                    LISTEN      32145/python3
```

可以看到，在终端 A中看到了在终端 B执行的Python命令进程，因为它们属于同一个network namespace，它们的网络信息是相互可见的。

另外也可以直接将一个指定的进程加入到一个 network namespace 中，如

```shell
$ nsenter --target 1234 --net=/var/run/netns/mynet
```

这里将 `pid=1234` 进程加入到 mynet 这个network namespace中。

注意：当一个进程执行结束时，此进程PID将自动从 mynet 中移除。如果要删除这个命名空间，必须将其相关的进程或资源全部关闭和释放。

```shell
$ ip netns pids mynet | xargs kill
$ ip netns del mynet
```

这里 kill 掉network namespace 中的所有进程ID，再将 network namespace 文件删除。

## 5. 总结

这里介绍了在 Linux 下如何创建一个 network namespace 文件，可以将这个文件其理解为一个进程组,。以及如何创建一个新的网络命名空间，将将其添加到一个进程组中。



# 源码实现

我们知道了网络命名空间的基本操作用法，下面我们接着源码看一下 `contained` 是如何实现这一块的。

跟踪函数  `netns.NewNetNS(netnsMountDir)` ->`NewNetNSFromPID()` -> `newNS(baseDir, pid)` ，实现源码

https://github.com/containerd/containerd/blob/5fdf55e493d68079d22a08de1a7afe522e5838e5/pkg/netns/netns_linux.go#L48-L139

```go
// Some of the following functions are migrated from
// https://github.com/containernetworking/plugins/blob/main/pkg/testutils/netns_linux.go

// newNS creates a new persistent (bind-mounted) network namespace and returns the
// path to the network namespace.
// If pid is not 0, returns the netns from that pid persistently mounted. Otherwise,
// a new netns is created.
func newNS(baseDir string, pid uint32) (nsPath string, err error) {
	b := make([]byte, 16)

	_, err = rand.Read(b)


  // 创建 network namespace 对应的目录和 mount point
	// Create the directory for mounting network namespaces
	// This needs to be a shared mountpoint in case it is mounted in to
	// other namespaces (containers)
	if err := os.MkdirAll(baseDir, 0755); err != nil {
		return "", err
	}

	// create an empty file at the mount point and fail if it already exists
	nsName := fmt.Sprintf("cni-%x-%x-%x-%x-%x", b[0:4], b[4:6], b[6:8], b[8:10], b[10:])
	nsPath = path.Join(baseDir, nsName)
	mountPointFd, err := os.OpenFile(nsPath, os.O_RDWR|os.O_CREATE|os.O_EXCL, 0666)
	if err != nil {
		return "", err
	}
	mountPointFd.Close()

	defer func() {
		// Ensure the mount point is cleaned up on errors
		if err != nil {
			os.RemoveAll(nsPath)
		}
	}()

  
  // 如果指定了pid，则返回其对应的 network namespace 文件
	if pid != 0 {
		procNsPath := getNetNSPathFromPID(pid)
    
    // 重点! 将 netns 挂载到 nsPath
		if err = unix.Mount(procNsPath, nsPath, "none", unix.MS_BIND, ""); err != nil {
			return "", fmt.Errorf("failed to bind mount ns src: %v at %s: %w", procNsPath, nsPath, err)
		}
		return nsPath, nil
	}

	var wg sync.WaitGroup
	wg.Add(1)

	// do namespace work in a dedicated goroutine, so that we can safely
	// Lock/Unlock OSThread without upsetting the lock/unlock state of
	// the caller of this function
	go (func() {
		defer wg.Done()
    
    // 1. 锁定goroutine 与 thread
		runtime.LockOSThread()

		var origNS cnins.NetNS
    // 2. getCurrentThreadNetNSPath()获取当前进程的main线程的netNSPath，然后调用 cnins.GetNS() 打开文件句柄
		origNS, err = cnins.GetNS(getCurrentThreadNetNSPath())
		if err != nil {
			return
		}
    // 最后，关闭 netns 打开的file句柄
		defer origNS.Close()

    // 3. 在当前线程创建新一个 netns
		// create a new netns on the current thread
		err = unix.Unshare(unix.CLONE_NEWNET)
		if err != nil {
			return
		}

    // 切换回原来的 netns
		defer origNS.Set()

    // 4. 将当前进程对应的新的 network namespace 挂载到 nsPath, 使命名空间永远存在，即使ns内没有线程
		// bind mount the netns from the current thread (from /proc) onto the
		// mount point. This causes the namespace to persist, even when there
		// are no threads in the ns.
		err = unix.Mount(getCurrentThreadNetNSPath(), nsPath, "none", unix.MS_BIND, "")

	})()
	wg.Wait()

	return nsPath, nil
}
```

实现整体流程：

1. 首先创建一个挂载点 `nsPath`,  这个挂载点对应的正是上面的 mynet
2. 如果当前pid 非0，则表示已经存在对应进程的netns, 则直接将其netns挂载点 nsPath 即可
3. 否则创建一个新的 netns, 并将其挂载到 nsPath ，实现将当前网络命名空间加入nsPath 这个network namespace `进程组`

这里重点看一下第三步。

1. 调用 `runtime.LockOSThread()` 锁定 `goroutine` 与 `OS Thread` 两者的绑定关系，这样可以防止 goroutine在执行过程中被调度器调度，使这个goroutine永远在这个thread运行，直到结束或提前返回。当goroutine结束时，`OS Thread` 将被操作系统自动回收，对其介绍参考 [《Golang中的runtime.LockOSThread 和 runtime.UnlockOSThread》](https://blog.haohtml.com/archives/30780/ )。
2. 调用 `getCurrentThreadNetNSPath()` 获取当前进程的main线程的netNSPath(换句话说获取当前进程的netns 文件路径)，然后调用 `cnins.GetNS()` 打开文件句柄
3. `unix.Unshare(unix.CLONE_NEWNET)` 在当前线程上创建一个新的 network namespace，此时环境已是最新的 network namespace
4. 通过`unix.Mount` 将当前 network namespace 挂载到（加入点） `nsPath`。这里挂载参数与Linux不一样，这里使用了 `/proc/%d/task/%d/ns/net` , 它对应的是进程当前的网络命名空间（自从 `unix.Unshare(unix.CLONE_NEWNET)` 被调用后，进程所属的网络命名空间就发生了变化 ），此命令会使命名空间持续存在，即使ns中没有线程。请注意ns持续存在的意义 ！！！
5. 清理工作：`defer origNS.Set()` 表示恢复切换回原来的netns，接着调用  `defer origNS.Close()` 关闭上面打开的文件句柄；

这里 `origNS.Set()` 实现源码

```go
func (ns *netNS) Set() error {
	...

	if err := unix.Setns(int(ns.Fd()), unix.CLONE_NEWNET); err != nil {
		return fmt.Errorf("Error switching to ns %v: %v", ns.file.Name(), err)
	}

	return nil
}
```

这里的  `unix.Setns()` 函数表示切换到的网络命名空间。`Setns` 函数共有两个参数：`int(ns.Fd())` 表示命名空间的文件描述符，`unix.CLONE_NEWNET` 表示要切换到的命名空间类型为 `网络命名空间`。

可以看到这里 `origNS` 对象主要是为了保存原来的现场，等创建新的netns 并挂载到 nsPath结束后，需要恢复现场，同时将对象创建时打开的一些文件资源进行释放。



# 总结

函数最终返回的是一个nsPath路径，它对应的是 `/var/run/netns/`目录里的一个文件，如 `/var/run/netns/cni-9c52cfcd-f64b-7ba1-f05d-eb19bb474119`。



# 参考资料

- https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/
- https://github.com/containernetworking/plugins/blob/main/pkg/testutils/netns_linux.go
- https://github.com/golang/go/wiki/LockOSThread
- https://blog.haohtml.com/archives/30780/
- [Linux ip netns 命令](https://codeleading.com/article/22754990848/)