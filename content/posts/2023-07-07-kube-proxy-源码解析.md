---
title: kube-proxy 源码解析
author: admin
type: post
date: 2023-07-07T09:20:43+00:00
url: /archives/33728
categories:
 - 程序开发
tags:
 - k8s
 - kube-proxy

---
k8s版本：v1.17.3

# 组件简介 

kube-proxy是Kubernetes中的一个核心组件之一，它提供了一个网络代理和负载均衡服务，用于将用户请求路由到集群中的正确服务。

kube-proxy的主要功能包括以下几个方面：

 1. 服务代理：kube-proxy会监听Kubernetes API服务器上的服务和端口，并将请求转发到相应的后端Pod。它通过在节点上创建iptables规则或使用IPVS（IP Virtual Server）进行负载均衡，以保证请求的正确路由。
 2. 负载均衡：当多个Pod实例对外提供相同的服务时，kube-proxy可以根据负载均衡算法将请求分发到这些实例之间，以达到负载均衡的目的。它可以基于轮询、随机、源IP哈希等算法进行负载均衡。
 3. 故障转移：如果某个Pod实例不可用，kube-proxy会检测到并将其自动从负载均衡轮询中移除，从而保证用户请求不会被转发到不可用的实例上。
 4. 会话保持（Session Affinity）：kube-proxy可以通过设置会话粘性（Session Affinity）来将同一客户端的请求转发到同一Pod实例，从而保持会话状态的一致性。
 5. 网络代理：kube-proxy还可以实现网络地址转换（NAT）和访问控制列表（ACL）等网络代理功能，通过为集群内的服务提供统一的入口地址和访问策略。

总之 `kube-proxy` 在Kubernetes集群中扮演着路由和负载均衡的重要角色，为集群内的服务提供可靠的网络连接和请求转发功能。

# 实现逻辑 

组件入口文件为 ` [cmd/kube-proxy/proxy.go](https://github.com/kubernetes/kubernetes/blob/v1.27.3/cmd/kube-proxy/proxy.go) `。

这里我们先介绍一下 `options` 这个数据结构，它主要有来存储一些配置项

```
type Options struct {
  // 配置文件路径
  ConfigFile string
​
// 将配置写入到文件
  WriteConfigTo string
​
  // bool值，如果为true,则删除所有iptables/ipvs 规则，然后退出程序
  CleanupAndExit bool

  // WindowsService should be set to true if kube-proxy is running as a service on Windows.
  // Its corresponding flag only gets registered in Windows builds
  WindowsService bool
​
  // KubeProxy configuration配置
  config *kubeproxyconfig.KubeProxyConfiguration
​
  // 监控配置文件对象，fsnotify
  watcher filesystem.FSWatcher

  // proxyServer is the interface to run the proxy server
  proxyServer proxyRun
  // errCh is the channel that errors will be sent
  errCh chan error
​
​
  // master is used to override the kubeconfig's URL to the apiserver.
  master string
  // 服务健康检查 healthServer. 端口号 10256
  healthzPort int32
  // 指标服务器 metrics server. 端口号 10249
  metricsPort int32
​
  // 如果从命令行标志设置，则优先于配置文件中的“HostnameOverride”值
  hostnameOverride string
}
```

最主要的几个字段

`ConfigFile` 配置文件路径

`WriteConfigTo` 将配置信息 [`Options.config`](https://github.com/kubernetes/kubernetes/blob/v1.27.3/cmd/kube-proxy/app/server.go#L101-L135) 写入到文件

`config` KubeProxy配置 ，对象 ` [*kubeproxyconfig.KubeProxyConfiguration](https://github.com/kubernetes/kubernetes/blob/v1.27.3/pkg/proxy/apis/config/types.go) `

`watcher` 一个fsnotify对象，用来监控配置文件 `ConfigFile` 内容变更

`proxyServer` proxy Server, 重要！这里是一个接口，它只有 `Run() error` 这一个方法。

后面会用到这个 options 的一些配置项，这里我们介绍一下每个字段的意义，对于后面理解有很帮助。

另外我们还需要关注另一个对象 `Proxier` 的数据结构, 它是一个基于iptables的代理，用于提供实际后端的服务之间的连接，所有 proxy rules 都是由它来完成的。由于字段比较的多，这里就不再一一列出，其声明见 [https://github.com/kubernetes/kubernetes/blob/v1.27.3/pkg/proxy/iptables/proxier.go#L154-L222](https://github.com/kubernetes/kubernetes/blob/v1.27.3/pkg/proxy/iptables/proxier.go#L154-L222)

注意：下面的代码根据重要性，可能会有所裁剪。

现在我们正式看一下组件 `kube proxy` 实现，入口函数为

```
func NewProxyCommand() *cobra.Command {
  // 准备：初始化配置选项 options
  opts := NewOptions()

  cmd := &cobra.Command{
    Use: "kube-proxy",
    Long: `The Kubernetes network proxy runs on each node. This with the apiserver API to configure the proxy.`,
    RunE: func(cmd *cobra.Command, args []string) error {
      verflag.PrintAndExitIfRequested()
      cliflag.PrintFlags(cmd.Flags())
​
      if err := initForOS(opts.WindowsService); err != nil {
        return fmt.Errorf("failed os init: %w", err)
      }
​
      // 一、初始化操作：
      // 1. watch监听配置文件的变更（相关字段 o.watcher）
      // 2. Healthz（端口10256） 和 metric 服务地址初始化（端口10249）
      if err := opts.Complete(); err != nil {
        return fmt.Errorf("failed complete: %w", err)
      }
​
      // 二、验证 kube-proxy的配置项 KubeProxyConfiguration
      if err := opts.Validate(); err != nil {
        return fmt.Errorf("failed validate: %w", err)
      }
      // add feature enablement metrics
      utilfeature.DefaultMutableFeatureGate.AddMetrics()
​
      // 三、启动服务
      if err := opts.Run(); err != nil {
        klog.ErrorS(err, "Error running ProxyServer")
        return err
      }
​
      return nil
    }
  }
​
  // 设置一些默认配置项
  var err error
  opts.config, err = opts.ApplyDefaults(opts.config)
​
  return cmd
}
```

主要从 `cobra`库的 `RunE()`函数入手。主要逻辑分三块：

 1. 创建一个监控配置文件 configFile 的对象，以后有变更的时候将自动收到相应的事件；同时还会初始化 healthServer 和 metricServer 的监听地址。
 2. 验证 `kube-proxy` 的配置项 _KubeProxyConfiguration_
 3. 启动服务

现在我们的重点是看一下服务启动的实现。

```
// cmd/kube-proxy/app/server.go
​
// Run runs the specified ProxyServer.
func (o *Options) Run() error {
  defer close(o.errCh)
​
  // 一、将配置 kube-proxy的配置  *config.KubeProxyConfiguration 写入文件（相关字段o.config）
  if len(o.WriteConfigTo) > 0 {
    return o.writeConfigFile()
  }
​
  // 二、清除所有ipables规则(ipv4和ipv6)
  if o.CleanupAndExit {
    return cleanupAndExit()
  }
​
  // 三、创建代理服务
  proxyServer, err := NewProxyServer(o)
  if err != nil {
    return err
  }
  // 启动代理服务
  o.proxyServer = proxyServer
  return o.runLoop()
}
```

可以看到，这里根据配置项来决定到底实现什么逻辑，但同时只能做一件事。

如果指定了 `Options.WriteConfigTo` 字段的话，则直接将配置信息写入到指定的文件，然后退出。

如果 `Options.CleanupAndExit` 为 `true` ，则删除掉所有 `iptables` 和 `ipvs` 规则，然后退出。

否则通过 `NewProxyServer()` 来创建一个proxier ，然后调用 `Options.runLoop()` 方法启动整个kubeProxy服务。

现在我们的关注重点又变成 `NewProxyServer()` 函数和 `o.runLoop()`方法了。

对于创建 proxyServer 实现上是通过 `newProxyServer()` 来实现的

```
// cmd/kube-proxy/app/server_others.go
​
func newProxyServer(
  config *proxyconfigapi.KubeProxyConfiguration,
  master string) (*ProxyServer, error) {
    // 创建一个 clientSet 对象，用于与 apiServer 通讯
    client, eventClient, err := createClients(config.ClientConnection, master)

    // New返回一个新接口，该接口将执行 os/exec 命令
    execer := exec.New()
   // New返回一个新接口，该接口将执行iptables
    iptInterface := utiliptables.New(execer, primaryProtocol)

    dualStack := true // While we assume that node supports, we do further checks below

    // 创建执行ipables 的 ipv4 和ipv6 接口
    if primaryProtocol == utiliptables.ProtocolIPv4 {
		ipt[0] = iptInterface
		ipt[1] = utiliptables.New(execer, utiliptables.ProtocolIPv6)
    } else {
		ipt[0] = utiliptables.New(execer, utiliptables.ProtocolIPv4)
		ipt[1] = iptInterface
    }

    // 代理模式，分 iptables 和 ipvs 两种
    if proxyMode == proxyconfigapi.ProxyModeIPTables {
      if dualStack {
        proxier, err = iptables.NewDualStackProxier(...)
      } else {
        proxier, err = iptables.NewProxier(...)
      }

    } else if proxyMode == proxyconfigapi.ProxyModeIPVS {
        kernelHandler := ipvs.NewLinuxKernelHandler()
        ipsetInterface = utilipset.New(execer)
        ipvsInterface = utilipvs.New()

        if dualStack {
          proxier, err = ipvs.NewDualStackProxier(...)

        } else {
          proxier, err = ipvs.NewProxier(...)
        }
    }

    // return
    return &ProxyServer{
      Client:                 client,
      EventClient:            eventClient,
      IptInterface:           iptInterface,
      IpvsInterface:          ipvsInterface,
      IpsetInterface:         ipsetInterface,
      execer:                 execer,
      Proxier:                proxier,
      Broadcaster:            eventBroadcaster,
      Recorder:               recorder,
      ConntrackConfiguration: config.Conntrack,
      Conntracker:            &realConntracker{},
      ProxyMode:              proxyMode,
      NodeRef:                nodeRef,
      MetricsBindAddress:     config.MetricsBindAddress,
      BindAddressHardFail:    config.BindAddressHardFail,
      EnableProfiling:        config.EnableProfiling,
      OOMScoreAdj:            config.OOMScoreAdj,
      ConfigSyncPeriod:       config.ConfigSyncPeriod.Duration,
      HealthzServer:          healthzServer,
      localDetectorMode:      detectLocalMode,
      podCIDRs:               podCIDRs,
    }, nil
}
```

通过调用 `utiliptables.New` 来返回一个执行 `iptables` 规则命令的实例 [runner](https://github.com/kubernetes/kubernetes/blob/v1.27.3/pkg/util/iptables/iptables.go#L203-L214)，其实现了 ` [Interface](https://github.com/kubernetes/kubernetes/blob/v1.27.3/pkg/util/iptables/iptables.go#L48-L97) ` 接口

```
// pkg/util/iptables/iptables.go#L48-L97
type Interface interface {
    EnsureChain(table Table, chain Chain) (bool, error)
    FlushChain(table Table, chain Chain) error
    DeleteChain(table Table, chain Chain) error
    ChainExists(table Table, chain Chain) (bool, error)
    EnsureRule(position RulePosition, table Table, chain Chain, args ...string) (bool, error)
    DeleteRule(table Table, chain Chain, args ...string) error
    IsIPv6() bool
    Protocol() Protocol
    SaveInto(table Table, buffer *bytes.Buffer) error
    Restore(table Table, data []byte, flush FlushFlag, counters RestoreCountersFlag) error
    RestoreAll(data []byte, flush FlushFlag, counters RestoreCountersFlag) error
    Monitor(canary Chain, tables []Table, reloadFunc func(), interval time.Duration, stopCh <-chan struct{})
    HasRandomFully() bool
    Present() bool
}
```

从接口方法声明可得知，这里包含了所有iptables的操作命令，包括备份 与恢复。

对于 `proxyServer` 的创建需要根据 `proxyMode` 来决定，同时还要考虑到 `Single-stack` 和 `dual-stack` 两种不同的工作模式。

> Single-stack和dual-stack是互联网协议 (IP) 网络中两种不同的工作模式。它们的区别在于支持的IP协议版本和网络地址类型。
>
>
> 1. Single-stack模式：
>    - Single-stack是指网络只支持一种IP协议版本，通常是IPv4（Internet Protocol version 4）。
>
>    - 在Single-stack模式中，所有网络设备和主机仅使用IPv4地址进行通信。
>
>    - 网络设备和主机只能使用IPv4协议进行路由和数据传输。
> 2. Dual-stack模式：
>    - Dual-stack是指网络同时支持两种IP协议版本，即IPv4和IPv6（Internet Protocol version 6）。
>
>    - 在Dual-stack模式中，网络设备和主机可以同时使用IPv4和IPv6地址进行通信。
>
>    - 网络设备和主机可以根据需要选择使用IPv4还是IPv6协议进行路由和数据传输。
>
> 主要区别如下：
>
>
> - IP协议版本：Single-stack只支持IPv4，而Dual-stack同时支持IPv4和IPv6。
>
> - 地址类型：Single-stack仅使用IPv4地址，而Dual-stack可以使用IPv4和IPv6地址。
>
> - 兼容性：Single-stack模式下，无法与IPv6-only设备进行通信；而Dual-stack模式下，可以与同时支持IPv4和IPv6的设备进行通信。
>
> - 未来发展：由于IPv4地址短缺和IPv6的广泛推广，Dual-stack模式被认为是未来网络发展的趋势。
>
>
> 在过渡期，许多网络同时支持Single-stack和Dual-stack，以确保既能与使用IPv4的设备进行通信，又能与使用IPv6的设备进行通信

由于 proxyServer 是重点，我们再看一下它的数据结构

```
// cmd/kube-proxy/app/server.go
​
type ProxyServer struct {
  // clientSet 对象，访问apiServer的客户端对象
  Client clientset.Interface
​
  EventClient v1core.EventsGetter
​
  // iptables/ipvs相关，一个是iptables 模式，另一个是 ipvs模式
  IptInterface   utiliptables.Interface // 执行ipables规则命令实现
  IpvsInterface  utilipvs.Interface // 执行ipvs命令实例
  IpsetInterface utilipset.Interface // 执行ipset命令实例
  // 一组  os/exec API
  execer exec.Interface
​
  // 重点！核心的执行对象
  Proxier proxy.Provider
​
  // 事件广播，接受事件并将其发送到  EventSink 中
  Broadcaster events.EventBroadcaster
  // 事件记录器
  Recorder events.EventRecorder
​
  // Linux 内核中参数 nf_conntrack_max 设置，并包对 conntrack 链接与关闭超时设置
  ConntrackConfiguration kubeproxyconfig.KubeProxyConntrackConfiguration
  Conntracker            Conntracker // if nil, ignored
​
  // 两种模式： iptables 和 ipvs，同时只能使用其中一种（每一种模式都需要考虑Single-stack和dual-stack 的情况）
  ProxyMode           kubeproxyconfig.ProxyMode
  NodeRef             *v1.ObjectReference
  MetricsBindAddress  string
  BindAddressHardFail bool
  EnableProfiling     bool
  OOMScoreAdj         *int32
  ConfigSyncPeriod    time.Duration
​
  // 服务健康检查
  HealthzServer healthcheck.ProxierHealthUpdater
​
  // 检查节点的本地流量
  localDetectorMode kubeproxyconfig.LocalMode
  podCIDRs          []string // only used for LocalModeNodeCIDR
}
```

字段解释

`Client` 表示 clientSet ，apiServer 通讯客户端

`IptInterface` iptables 实现

`execer` 是_一组_ `os/exec API`

`Proxier` 重点！核心的执行对象 proxy

`Broadcaster` 事件广播，接受事件并将其发送到* _EventSink_ *中

`Recorder` 事件记录器

`ConntrackConfiguration` _conntrack_ 超时配置项

`Conntracker` 接口对象，也是 _conntrack_ 相关

`ProxyMode` 两种模式： _iptables_ _和_ _ipvs_，同时只能使用其中一种

`localDetectorMode` 从节点中检查本地流量，目前支持四种模式：`ClusterCIDR`/`NodeCIDR`/`BridgeInterface`/`InterfaceNamePrefix`

`podCIDRs` 网络 cidr

我们再看一下 `runLoop()` 的实现

```
func (o *Options) runLoop() error {
  // 1. 启动 fileWatcher 服务，监控配置文件的变更, 参考 pkg/util/filesystem/watcher.go#L73
  if o.watcher != nil {
    o.watcher.Run()
  }
​
  // 2. 启动proxy服务
  // run the proxy in goroutine
  go func() {
    err := o.proxyServer.Run()
    o.errCh <- err
  }()
​
  for {
    err := <-o.errCh
    if err != nil {
      return err
    }
  }
}
```

函数 `o.proxyServer.Run()`的实现。

```
// cmd/kube-proxy/app/server.go
​
func (s *ProxyServer) Run() error {
​
  if s.Broadcaster != nil && s.EventClient != nil {
    stopCh := make(chan struct{})
    s.Broadcaster.StartRecordingToSink(stopCh)
  }
​
  // 1. 启用HealthzServer 和 MetricsServer 服务
  // Start up a healthz server if requested
  serveHealthz(s.HealthzServer, errCh)
​
  // Start up a metrics server if requested
  serveMetrics(s.MetricsBindAddress, s.ProxyMode, s.EnableProfiling, errCh)
​
  // 2. 配置iptables 中的 Conntracker 相关参数
  // 设置Linux内核中的 nf_conntrack_max 参数，控制跟踪表（Connection Tracking Table）中的最大连接数
  // 设置Conntracker的TCP连接与关闭超时时间
  // Tune conntrack, if requested
  // Conntracker is always nil for windows
  if s.Conntracker != nil {
    max, err := getConntrackMax(s.ConntrackConfiguration)
    if err != nil {
      return err
    }
    if max > 0 {
      err := s.Conntracker.SetMax(max)
      if err != nil {
        if err != errReadOnlySysFS {
          return err
        }

        const message = "CRI error: /sys is read-only: " +
          "cannot modify conntrack limits, problems may arise later (If running Docker, see docker issue #24000)"
        s.Recorder.Eventf(s.NodeRef, nil, api.EventTypeWarning, err.Error(), "StartKubeProxy", message)
      }
    }
​
    if s.ConntrackConfiguration.TCPEstablishedTimeout != nil && s.ConntrackConfiguration.TCPEstablishedTimeout.Duration > 0 {
      timeout := int(s.ConntrackConfiguration.TCPEstablishedTimeout.Duration / time.Second)
      if err := s.Conntracker.SetTCPEstablishedTimeout(timeout); err != nil {
        return err
      }
    }
​
    if s.ConntrackConfiguration.TCPCloseWaitTimeout != nil && s.ConntrackConfiguration.TCPCloseWaitTimeout.Duration > 0 {
      timeout := int(s.ConntrackConfiguration.TCPCloseWaitTimeout.Duration / time.Second)
      if err := s.Conntracker.SetTCPCloseWaitTimeout(timeout); err != nil {
        return err
      }
    }
  }
​
  noProxyName, err := labels.NewRequirement(apis.LabelServiceProxyName, selection.DoesNotExist, nil)
  if err != nil {
    return err
  }
​
  noHeadlessEndpoints, err := labels.NewRequirement(v1.IsHeadlessService, selection.DoesNotExist, nil)
  if err != nil {
    return err
  }
​
  labelSelector := labels.NewSelector()
  labelSelector = labelSelector.Add(*noProxyName, *noHeadlessEndpoints)
​
  // 3. 创建一个 informers 服务,指定 options.LabelSelector；参考 https://blog.haohtml.com/archives/32179
  // RegisterHandler()调用需要在创建源之前进行，因为源只会通知更改，如果还没有注册处理程序，则初始更新（在进程启动时）可能会丢失。
​
  // Make informers that filter out objects that want a non-default service proxy.
  informerFactory := informers.NewSharedInformerFactoryWithOptions(s.Client, s.ConfigSyncPeriod,
    informers.WithTweakListOptions(func(options *metav1.ListOptions) {
      options.LabelSelector = labelSelector.String()
    }))
​
  // Create configs (i.e. Watches for Services and EndpointSlices)
  // Note: RegisterHandler() calls need to happen before creation of Sources because sources
  // only notify on changes, and the initial update (on process start) may be lost if no handlers
  // are registered yet.
​
  // 注册 Service 和 endpointSlice 对象变更事件，服务启动时，需要强制调用一次 cache.WaitForNamedCacheSync() 来进行cache同步
  // 3.1 serviceConfig
  serviceConfig := config.NewServiceConfig(informerFactory.Core().V1().Services(), s.ConfigSyncPeriod)
  serviceConfig.RegisterEventHandler(s.Proxier)  // 注册对象变更事件
  go serviceConfig.Run(wait.NeverStop)
​
  // 3.2 endpointSliceConfig
  endpointSliceConfig := config.NewEndpointSliceConfig(informerFactory.Discovery().V1().EndpointSlices(), s.ConfigSyncPeriod)
  endpointSliceConfig.RegisterEventHandler(s.Proxier) // 注册对象变更事件
  go endpointSliceConfig.Run(wait.NeverStop)
​
  // This has to start after the calls to NewServiceConfig because that
  // function must configure its shared informer event handlers first.
  // 启动 informer 服务
  informerFactory.Start(wait.NeverStop)
​
  // Make an informer that selects for our nodename.
  // 4. 创建一个 node informer, 指定 options.FieldSelector
  currentNodeInformerFactory := informers.NewSharedInformerFactoryWithOptions(s.Client, s.ConfigSyncPeriod,
    informers.WithTweakListOptions(func(options *metav1.ListOptions) {
      options.FieldSelector = fields.OneTermEqualSelector("metadata.name", s.NodeRef.Name).String()
    }))
  nodeConfig := config.NewNodeConfig(currentNodeInformerFactory.Core().V1().Nodes(), s.ConfigSyncPeriod)
  if s.localDetectorMode == kubeproxyconfig.LocalModeNodeCIDR {
    nodeConfig.RegisterEventHandler(proxy.NewNodePodCIDRHandler(s.podCIDRs))
  }
  nodeConfig.RegisterEventHandler(s.Proxier) // 注册对象变更事件
  go nodeConfig.Run(wait.NeverStop)
  currentNodeInformerFactory.Start(wait.NeverStop)
​
​
  s.birthCry()
​
  // 5. 如果为proxyMode == iptables的话，则调用的是 pkg/proxy/ipables/proxier.go#L509
  go s.Proxier.SyncLoop()
​
  return <-errCh
}
```

大概可以分以下几个步骤

## 启动 _HealthzServer_ 和 _MetricsServer_ 服务 

一个是服务健康检测服务，另一个是指标服务

## 配置 iptables 中的 Conntracker 相关参数 

设置 `conntracker` 数量，并设置其建立连接、关闭连接超时时间

## 创建 informers，注册 service 和 endpointSlice 变更事件 

创建一个 _informers_ 实例(指定 options.LabelSelector), 并注册 `Service` 和 `endpointSlice` 对象内容变更事件，最后调用 `serviceConfig.Run(wait.NeverStop)` 启用informers 服务，由于是刚初始化，因此需要调用 一次 `OnServiceSynced()`这个方法。我们知道 `informer` 是 `client-go` 库其中的一部分，对于整个 `client-go` 的介绍请参考 。这里以 Service 为例，介绍一下 informers 的处理。**3.1** 首先调用 `NewServiceConfig()` 初始化一个 `ServiceConfig` 服务

```
// pkg/proxy/config/config.go
type ServiceConfig struct {
    listerSynced  cache.InformerSynced
    eventHandlers []ServiceHandler
}

func NewServiceConfig(serviceInformer coreinformers.ServiceInformer, resyncPeriod time.Duration) *ServiceConfig {
  // 创建对象, 此时还没有任何 eventHandler
	result := &ServiceConfig{
		listerSynced: serviceInformer.Informer().HasSynced,
	}

// 注册所有事件到 informer 中,对应 client-go 框架图中的步骤六
	serviceInformer.Informer().AddEventHandlerWithResyncPeriod(
		cache.ResourceEventHandlerFuncs{
			AddFunc:    result.handleAddService,
			UpdateFunc: result.handleUpdateService,
			DeleteFunc: result.handleDeleteService,
		},
		resyncPeriod,
	)

	return result
}
```

这里注册了一些 cache 事件处理函数，对应的是  文章中的步骤六，其为CRD实现中的一部分。

**3.2** 接着调用 `serviceConfig.RegisterEventHandler(s.Proxier)` 注册事件处理handler

```
// RegisterEventHandler registers a handler which is called on every service change.
func (c *ServiceConfig) RegisterEventHandler(handler ServiceHandler) {
	c.eventHandlers = append(c.eventHandlers, handler)
}
```

这里函数 `RegisteEventHandler(handler ServiceHandler)` 的参数是 ` [ServiceHandler](https://github.com/kubernetes/kubernetes/blob/v1.27.3/pkg/proxy/config/config.go#L32-L47) ` 接口，其定义

```
// pkg/proxy/config/config.go
type ServiceHandler interface {
	// 只要观察到新服务对象的创建，就会调用OnServiceAdd
	OnServiceAdd(service *v1.Service)
	OnServiceUpdate(oldService, service *v1.Service)
	OnServiceDelete(service *v1.Service)

	// 一旦调用了所有初始事件处理程序并且状态完全传播到本地缓存，就会调用OnServiceSynced
	OnServiceSynced()
}
```

一共四个方法，不同事件对应不同的处理方法。如添加一个新的Service对象时，将调用 `OnServiceAdd`, 而更新一个Service 对象则对应的是 `OnServiceUpdate`，删除一个Service对应的是 `OnServiceDelete`。注意：对于 `OnServiceSynced()` 的使用场景有点不一样。

**3.3** 最后调用 `go serviceConfig.Run(wait.NeverStop)` 启动服务。

```
// pkg/proxy/config/config.go
// Run waits for cache synced and invokes handlers after syncing.
func (c *ServiceConfig) Run(stopCh <-chan struct{}) {

	// 强制同步cache
	if !cache.WaitForNamedCacheSync("service config", stopCh, c.listerSynced) {
		return
	}

	for i := range c.eventHandlers {
		c.eventHandlers[i].OnServiceSynced()
	}
}
```

可以看到这里必须先强制同步一下 `cache`, 然后再调用 `OnServiceSynced()` 方法，在本例中实际上执行是 ` [s.Proxier.OnServiceSynced()](https://github.com/kubernetes/kubernetes/blob/v1.27.3/pkg/proxy/iptables/proxier.go#L553-L563) ` 方法。

```
// pkg/proxy/iptables/proxier.go#L553

func (proxier *Proxier) OnServiceSynced() {
	proxier.mu.Lock()
	proxier.servicesSynced = true
	proxier.setInitialized(proxier.endpointSlicesSynced)
	proxier.mu.Unlock()

	// Sync unconditionally - this is called once per lifetime.
	proxier.syncProxyRules()
}
```

这里的 `proxies.syncProxyRules()`主要用来维护 iptables 规则，如创建一个Service，它则会在本机写一条iptables 规则，后面我们再单独介绍它。对于`endpointSlice` 对象与 Service 的机制完全一样，只不过它实现的是 ` [EndpointSliceHandler](https://github.com/kubernetes/kubernetes/blob/v1.27.3/pkg/proxy/config/config.go#L49-L64) ` 接口, 定义如下

```
// pkg/proxy/config/config.go
type EndpointSliceHandler interface {
	OnEndpointSliceAdd(endpointSlice *discovery.EndpointSlice)
	OnEndpointSliceUpdate(oldEndpointSlice, newEndpointSlice *discovery.EndpointSlice)
	OnEndpointSliceDelete(endpointSlice *discovery.EndpointSlice)

	OnEndpointSlicesSynced()
}
```

## 创建 `node informers` ，注册节点变更事件 

创建一个 `node informers` 实例，注册节点信息变更事件。其事件处理handler 实现的是一个 ` [NodeHandler](https://github.com/kubernetes/kubernetes/blob/v1.27.3/pkg/proxy/config/config.go#L248-L263) ` 接口

```
// pkg/proxy/config/config.go
type NodeHandler interface {
	OnNodeAdd(node *v1.Node)
	OnNodeUpdate(oldNode, node *v1.Node)
	OnNodeDelete(node *v1.Node)

	OnNodeSynced()
}
```


​与 Service 的处理机制完全一样。

## 启动 proxier.SyncLoop 服务 

开启一个goroutine，执行 `s.Proxier.SyncLoop()` 服务，这里的 `s.Proxier` 对象是根据 `proxyMode` 而定。对于这一步可以简单的理解为启动了一个 Loop 服务。对于 `iptables` 模式来讲，对应的是 `pkg/proxy/ipables/proxier.go#L509`，调用流程如下：

```
// pkg/proxy/iptables/proxier.go#L509
func (proxier *Proxier) SyncLoop() {
	if proxier.healthzServer != nil {
		proxier.healthzServer.Updated()
	}

	proxier.syncRunner.Loop(wait.NeverStop)
}
```

```
// pkg/util/async/bounded_frequency_runner.go#L189
func (bfr *BoundedFrequencyRunner) Loop(stop <-chan struct{}) {
    bfr.timer.Reset(bfr.maxInterval)
    for {
      select {
        case <-stop:
            bfr.stop()
            klog.V(3).Infof("%s Loop stopping", bfr.name)
            return
        case <-bfr.timer.C():
            bfr.tryRun()
        case <-bfr.run:
            bfr.tryRun()
        case <-bfr.retry:
            bfr.doRetry()
        }
    }
}
```

总结

在第四步首先注册资源对象变更处理 Handler（`OnXXXAdd/OnXXXUpdate/OnXXXDelete` ），这样就可以实现当资源对象发生变化时，执行这些事件处理函数，其中这些事件处理函数内部将调用 `proxier.Sync()` 随后再向一个 channel (`bfr.run`)发送一个变更信号。等注册完事件 Handler 后，再调用 `.Run()` 函数（内部调用`OnXxxSynced`）将 `proxy rules` 规则写入本地。![](https://blogstatic.haohtml.com/uploads/2023/07/ff40094cefe93bdc5b198b7b8149f70d.png)

紧接着在第五步中启动一个 `proxiers.Loop` 服务，监听这个channel (`bfr.run`)并做出相应的处理响应 ` [bfr.tryRun()](https://github.com/kubernetes/kubernetes/blob/v1.27.3/pkg/util/async/bounded_frequency_runner.go#L189-L208) `，其将自动执行前面的自定义函数 `bfr.fn()`, 在本示例中也就是 ` [proxier.syncProxyRules()](https://github.com/kubernetes/kubernetes/blob/v1.27.3/pkg/proxy/iptables/proxier.go#L805-L1641) ` 函数 ，其实现代码比较的多也有一些复杂，不过对于本篇来说只要理解它的大概流程即可。

可以看到，对于我们指定的资源对象 Service/endpointSlice （informerFactory实例）和 node（currentNodeInformerFactory实例），只要发生变更将自动触发调用函数`proxier.syncProxyRules()` ，将 proxy rules 规则写入本地，这样当本地有流量经过时，这些 proxy rules 将生效，将流量代理到指定的endpoint。

# 总结 

`kube proxy` 组件通过监听一些资源的变更事件，来实现对 `proxy rules` 规则的维护，以此实现对 k8s Service 流量的管理。

另外对于如果从 `api-server` 监听资源的变更这里暂未介绍，有兴趣的话可以关注一下 `informers` 。

# 参考资料 

 * [https://github.com/kubernetes/kubernetes/blob/v1.27.3/cmd/kube-proxy/proxy.go](https://github.com/kubernetes/kubernetes/blob/v1.27.3/cmd/kube-proxy/proxy.go)
 * [https://blog.haohtml.com/archives/32179](https://blog.haohtml.com/archives/32179)
 * [https://kubernetes.io/zh-cn/docs/concepts/services-networking/dual-stack/](https://kubernetes.io/zh-cn/docs/concepts/services-networking/dual-stack/)
 * [https://kubernetes.io/zh-cn/docs/concepts/services-networking/endpoint-slices/](https://kubernetes.io/zh-cn/docs/concepts/services-networking/endpoint-slices/)
 * [https://time.geekbang.org/column/article/68636](https://time.geekbang.org/column/article/68636)