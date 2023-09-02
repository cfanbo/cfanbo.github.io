---
title: k8s之kube-controller-manager 源码分析
author: admin
type: post
date: 2023-08-01T11:24:43+00:00
url: /archives/34724
categories:
 - 程序开发
tags:
 - k8s

---
Kubernetes 控制器管理器（`kube-controller-manager`）是一个守护进程，内嵌随 Kubernetes 一起发布的核心控制回路。 在机器人和自动化的应用中，控制回路是一个永不休止的循环，用于调节系统状态。 在 Kubernetes 中，每个控制器是一个控制回路，通过 API 服务器监视集群的共享状态， 并尝试进行更改以将当前状态转为期望状态。 目前，Kubernetes 自带的控制器例子包括副本控制器、节点控制器、命名空间控制器和服务账号控制器等。

本文不对 `kube-controller-manager` 管理的每个控制器的执行原理做介绍，只是从全局观看一下kube-controller-manager 启动每个控制器的整体实现过程。

k8s: v1.27.3

文件： [cmd/kube-controller-manager/app/controllermanager.go][1]

# 控制器选项初始化 

```
// cmd/kube-controller-manager/app/controllermanager.go#L104
func NewControllerManagerCommand() *cobra.Command {

        // 所有内置控制器配置
        s, err := options.NewKubeControllerManagerOptions()
}
```

调用 `options.NewKubeControllerManagerOptions()` 对所有内置控制器及全局配置进行初始化。

```
func NewKubeControllerManagerOptions() (*KubeControllerManagerOptions, error) {
    componentConfig, err := NewDefaultComponentConfig()
    if err != nil {
        return nil, err
    }

    s := KubeControllerManagerOptions{
        Generic:         cmoptions.NewGenericControllerManagerConfigurationOptions(&componentConfig.Generic),
        KubeCloudShared: cpoptions.NewKubeCloudSharedOptions(&componentConfig.KubeCloudShared),
        ServiceController: &cpoptions.ServiceControllerOptions{
            ServiceControllerConfiguration: &componentConfig.ServiceController,
        },
        AttachDetachController: &AttachDetachControllerOptions{
            &componentConfig.AttachDetachController,
        },
        CSRSigningController: &CSRSigningControllerOptions{
            &componentConfig.CSRSigningController,
        },
        DaemonSetController: &DaemonSetControllerOptions{
            &componentConfig.DaemonSetController,
        },
        DeploymentController: &DeploymentControllerOptions{
            &componentConfig.DeploymentController,
        },
        StatefulSetController: &StatefulSetControllerOptions{
            &componentConfig.StatefulSetController,
        },
        DeprecatedFlags: &DeprecatedControllerOptions{
            &componentConfig.DeprecatedController,
        },
        EndpointController: &EndpointControllerOptions{
            &componentConfig.EndpointController,
        },
        EndpointSliceController: &EndpointSliceControllerOptions{
            &componentConfig.EndpointSliceController,
        },
        EndpointSliceMirroringController: &EndpointSliceMirroringControllerOptions{
            &componentConfig.EndpointSliceMirroringController,
        },
        EphemeralVolumeController: &EphemeralVolumeControllerOptions{
            &componentConfig.EphemeralVolumeController,
        },
        GarbageCollectorController: &GarbageCollectorControllerOptions{
            &componentConfig.GarbageCollectorController,
        },
        HPAController: &HPAControllerOptions{
            &componentConfig.HPAController,
        },
        JobController: &JobControllerOptions{
            &componentConfig.JobController,
        },
        CronJobController: &CronJobControllerOptions{
            &componentConfig.CronJobController,
        },
        NamespaceController: &NamespaceControllerOptions{
            &componentConfig.NamespaceController,
        },
        NodeIPAMController: &NodeIPAMControllerOptions{
            &componentConfig.NodeIPAMController,
        },
        NodeLifecycleController: &NodeLifecycleControllerOptions{
            &componentConfig.NodeLifecycleController,
        },
        PersistentVolumeBinderController: &PersistentVolumeBinderControllerOptions{
            &componentConfig.PersistentVolumeBinderController,
        },
        PodGCController: &PodGCControllerOptions{
            &componentConfig.PodGCController,
        },
        ReplicaSetController: &ReplicaSetControllerOptions{
            &componentConfig.ReplicaSetController,
        },
        ReplicationController: &ReplicationControllerOptions{
            &componentConfig.ReplicationController,
        },
        ResourceQuotaController: &ResourceQuotaControllerOptions{
            &componentConfig.ResourceQuotaController,
        },
        SAController: &SAControllerOptions{
            &componentConfig.SAController,
        },
        TTLAfterFinishedController: &TTLAfterFinishedControllerOptions{
            &componentConfig.TTLAfterFinishedController,
        },
        SecureServing:  apiserveroptions.NewSecureServingOptions().WithLoopback(),
        Authentication: apiserveroptions.NewDelegatingAuthenticationOptions(),
        Authorization:  apiserveroptions.NewDelegatingAuthorizationOptions(),
        Metrics:        metrics.NewOptions(),
        Logs:           logs.NewOptions(),
    }

    s.Authentication.RemoteKubeConfigFileOptional = true
    s.Authorization.RemoteKubeConfigFileOptional = true

    // Set the PairName but leave certificate directory blank to generate in-memory by default
    s.SecureServing.ServerCert.CertDirectory = ""
    s.SecureServing.ServerCert.PairName = "kube-controller-manager"
    s.SecureServing.BindPort = ports.KubeControllerManagerPort

    gcIgnoredResources := make([]garbagecollectorconfig.GroupResource, 0, len(garbagecollector.DefaultIgnoredResources()))
    for r := range garbagecollector.DefaultIgnoredResources() {
        gcIgnoredResources = append(gcIgnoredResources, garbagecollectorconfig.GroupResource{Group: r.Group, Resource: r.Resource})
    }

    s.GarbageCollectorController.GCIgnoredResources = gcIgnoredResources
    s.Generic.LeaderElection.ResourceName = "kube-controller-manager"
    s.Generic.LeaderElection.ResourceNamespace = "kube-system"

    return &s, nil
}
```

这里的许多控制器都做并发数据控制，如 `DeploymentController` 、`DaemonSetController` 和 `EndpointController` 等。

# 获取内置控制器 

```
func NewControllerManagerCommand() *cobra.Command {
    ...

        RunE: func(cmd *cobra.Command, args []string) error {
      ...

            // 2. 调用函数 KnownControllers()返回所有内置控制器名称，然后调用 s.Config() 获取一些通讯信息
            c, err := s.Config(KnownControllers(), ControllersDisabledByDefault.List())

        }
    }
```

首先调用 `KnownControllers()` 获取内置控制器

> 调用链为： `KnownControllers()` -> `sets.StringKeySet(NewControllerInitializers(IncludeCloudLoops))` -> `NewControllerInitializers()`

```
func KnownControllers() []string {
    // IncludeCloudLoops 表示 kube控制器管理器包括依赖于云提供商的控制器循环
    ret := sets.StringKeySet(NewControllerInitializers(IncludeCloudLoops))
    ret.Insert(
       saTokenControllerName,
    )

    return ret.List()
}
```

调用 `NewControllerInitializers()` 函数获取所有内置控制器

```
// cmd/kube-controller-manager/app/controllermanager.go#L429
func NewControllerInitializers(loopMode ControllerLoopMode) map[string]InitFunc {
    controllers := map[string]InitFunc{}

    // 所有内置控制器
    // All of the controllers must have unique names, or else we will explode.
    register := func(name string, fn InitFunc) {
        if _, found := controllers[name]; found {
            panic(fmt.Sprintf("controller name %q was registered twice", name))
        }
        controllers[name] = fn
    }

    register("endpoint", startEndpointController)
    register("endpointslice", startEndpointSliceController)
    register("endpointslicemirroring", startEndpointSliceMirroringController)
    register("replicationcontroller", startReplicationController)
    register("podgc", startPodGCController)
    register("resourcequota", startResourceQuotaController)
    register("namespace", startNamespaceController)
    register("serviceaccount", startServiceAccountController)
    register("garbagecollector", startGarbageCollectorController)
    register("daemonset", startDaemonSetController)
    register("job", startJobController)
    register("deployment", startDeploymentController)
    register("replicaset", startReplicaSetController)
    register("horizontalpodautoscaling", startHPAController)
    register("disruption", startDisruptionController)
    register("statefulset", startStatefulSetController)
    register("cronjob", startCronJobController)
    register("csrsigning", startCSRSigningController)
    register("csrapproving", startCSRApprovingController)
    register("csrcleaner", startCSRCleanerController)
    register("ttl", startTTLController)
    register("bootstrapsigner", startBootstrapSignerController)
    register("tokencleaner", startTokenCleanerController)
    register("nodeipam", startNodeIpamController)
    register("nodelifecycle", startNodeLifecycleController)
    if loopMode == IncludeCloudLoops {
        register("service", startServiceController)
        register("route", startRouteController)
        register("cloud-node-lifecycle", startCloudNodeLifecycleController)
        // TODO: volume controller into the IncludeCloudLoops only set.
    }
    register("persistentvolume-binder", startPersistentVolumeBinderController)
    register("attachdetach", startAttachDetachController)
    register("persistentvolume-expander", startVolumeExpandController)
    register("clusterrole-aggregation", startClusterRoleAggregrationController)
    register("pvc-protection", startPVCProtectionController)
    register("pv-protection", startPVProtectionController)
    register("ttl-after-finished", startTTLAfterFinishedController)
    register("root-ca-cert-publisher", startRootCACertPublisher)
    register("ephemeral-volume", startEphemeralVolumeController)
    if utilfeature.DefaultFeatureGate.Enabled(genericfeatures.APIServerIdentity) &&
        utilfeature.DefaultFeatureGate.Enabled(genericfeatures.StorageVersionAPI) {
        register("storage-version-gc", startStorageVersionGCController)
    }
    if utilfeature.DefaultFeatureGate.Enabled(kubefeatures.DynamicResourceAllocation) {
        register("resource-claim-controller", startResourceClaimController)
    }

    return controllers
}
```

这些内置控制器定义在`cmd/kube-controller-manager/app/`目录下。

 1. `podgc` :  Pod 垃圾回收控制器负责监控和清理没有被使用的 Pod。
 2. `nodelifecycle` : 节点生命周期控制器负责监控集群中节点的状态，并在节点出现故障或不可用时进行相应的处理，例如终止节点上的容器，或迁移工作负载到其他健康节点上。
 3. `replicaset`: ReplicaSet 控制器用于确保指定的 Pod 副本数处于运行状态，并在需要时创建或删除 Pod 来维持所需的副本数。
 4. `resourcequota`: 资源配额控制器用于限制命名空间中的资源使用量，以防止资源超额使用。

> 注意：对于pod 的创建是由 pod controller来完成的，根据架构设计，它应该位于 kubelet 组件中，见 `/pkg/kubelet/pod/` 目录。参考文章：https://blog.haohtml.com/archives/33163

接着调用 `s.Config()` 函数，生成控制器管理配置项，获取与集群通讯所必须的基本信息。

```
// Config return a controller manager config objective
func (s KubeControllerManagerOptions) Config(allControllers []string, disabledByDefaultControllers []string) (*kubecontrollerconfig.Config, error) {
    if err := s.Validate(allControllers, disabledByDefaultControllers); err != nil {
       return nil, err
    }

    if err := s.SecureServing.MaybeDefaultWithSelfSignedCerts("localhost", nil, []net.IP{netutils.ParseIPSloppy("127.0.0.1")}); err != nil {
       return nil, fmt.Errorf("error creating self-signed certificates: %v", err)
    }

    // 1. kubeconfig
    kubeconfig, err := clientcmd.BuildConfigFromFlags(s.Master, s.Generic.ClientConnection.Kubeconfig)
    if err != nil {
       return nil, err
    }
    kubeconfig.DisableCompression = true
    kubeconfig.ContentConfig.AcceptContentTypes = s.Generic.ClientConnection.AcceptContentTypes
    kubeconfig.ContentConfig.ContentType = s.Generic.ClientConnection.ContentType
    kubeconfig.QPS = s.Generic.ClientConnection.QPS
    kubeconfig.Burst = int(s.Generic.ClientConnection.Burst)

    // 2. clientSet
    client, err := clientset.NewForConfig(restclient.AddUserAgent(kubeconfig, KubeControllerManagerUserAgent))
    if err != nil {
       return nil, err
    }

    // 3. event
    eventBroadcaster := record.NewBroadcaster()
    eventRecorder := eventBroadcaster.NewRecorder(clientgokubescheme.Scheme, v1.EventSource{Component: KubeControllerManagerUserAgent})

    // 4. 控制器配置
    c := &kubecontrollerconfig.Config{
       Client:           client,
       Kubeconfig:       kubeconfig,
       EventBroadcaster: eventBroadcaster,
       EventRecorder:    eventRecorder,
    }
    if err := s.ApplyTo(c); err != nil {
       return nil, err
    }
    s.Metrics.Apply()

    return c, nil
}
```

主要是客户端通讯必须的 `client`，还一个集群通讯必须的 `kubeconfig`, 这两个应该比较的熟悉了，注意它们之间的一些区别。

客户端`client` 和 `kubeconfig` 是 Kubernetes 中两个不同的配置概念:

 * 客户端client 指的是用来访问 Kubernetes API 的客户端库,如 `client-go` 等。这些客户端需要指定访问 `API server` 的方式,如 URL、证书等。
 * kubeconfig 是 Kubernetes 客户端的通用配置文件,用于定义集群访问信息。它包含访问 API server 所需的所有信息,如 URL、证书、用户名等。kubeconfig 允许在同一台机器上访问多个 Kubernetes 集群。

主要区别:

 * 客户端(client)是访问 Kubernetes API 的代码库/工具。kubeconfig 更像是一个配置文件。
 * 客户端需要指定访问集群的信息,这些信息通常从 kubeconfig 文件中读取。也就是说 kubeconfig 提供了客户端所需的集群访问配置。
 * kubeconfig 包含所有集群的访问配置,支持在不同集群之间切换。而客户端只包含访问当前集群的配置。
 * 开发 Kubernetes 客户端需要实现客户端代码。而使用 kubeconfig 则通过 kubectl 工具就能访问 Kubernetes 集群,kubeconfig 为客户端提供了便利。

综上, kubeconfig 是配置文件,包含集群访问设置,为客户端程序提供集群访问所需的配置信息。客户端则是使用这些配置访问 Kubernetes API 的代码。

# 控制器执行 

对于控制器的执行调用入口函数 `Run()` 实现

```
// cmd/kube-controller-manager/app/controllermanager.go#L104
func NewControllerManagerCommand() *cobra.Command {

            // 3. 启动kube-controller-manager服务
            return Run(context.Background(), c.Complete())

}
```

参数 `c.Complete()` 指前面配置选项。

`Run()` 函数首先创建一个控制器上下文配置选项，用来在每个控制器执行时提供公共配置，然后调用 `StartConterllers()` 函数来启动所有的内置控制器。

```
// cmd/kube-controller-manager/app/controllermanager.go#L180
// Run runs the KubeControllerManagerOptions.
func Run(ctx context.Context, c *config.CompletedConfig) error {

    run := func(ctx context.Context, startSATokenController InitFunc, initializersFunc ControllerInitializersFunc) {
      // 创建控制器上下文配置选项
        controllerContext, err := CreateControllerContext(logger, c, rootClientBuilder, clientBuilder, ctx.Done())

        // 获取并执行所有控制器
        controllerInitializers := initializersFunc(controllerContext.LoopMode)
        if err := StartControllers(ctx, controllerContext, startSATokenController, controllerInitializers, unsecuredMux, healthzHandler); err != nil {
            logger.Error(err, "Error starting controllers")
            klog.FlushAndExit(klog.ExitFlushTimeout, 1)
        }

        controllerContext.InformerFactory.Start(stopCh)
        controllerContext.ObjectOrMetadataInformerFactory.Start(stopCh)
        close(controllerContext.InformersStarted)

        <-ctx.Done()
    }

    // 没有选举leader，直接启动所有 controllers 服务
    if !c.ComponentConfig.Generic.LeaderElection.LeaderElect {
        // 这里参数 NewControllerInitializers 就是上我们上面介绍过的注册所有控制器的函数是一个函数
        run(ctx, saTokenControllerInitFunc, NewControllerInitializers)
        return nil
    }

    // 有选举leader
    go leaderElectAndRun(ctx, c, id, electionChecker,
        c.ComponentConfig.Generic.LeaderElection.ResourceLock,
        c.ComponentConfig.Generic.LeaderElection.ResourceName,
        leaderelection.LeaderCallbacks{
            OnStartedLeading: func(ctx context.Context) {
                initializersFunc := NewControllerInitializers
                if leaderMigrator != nil {
                    // If leader migration is enabled, we should start only non-migrated controllers
                    //  for the main lock.
                    initializersFunc = createInitializersFunc(leaderMigrator.FilterFunc, leadermigration.ControllerNonMigrated)
                    logger.Info("leader migration: starting main controllers.")
                }

                // 启动所有 controllers 服务
                run(ctx, startSATokenController, initializersFunc)
            },
            OnStoppedLeading: func() {
                logger.Error(nil, "leaderelection lost")
                klog.FlushAndExit(klog.ExitFlushTimeout, 1)
            },
        })
}
```

从`StartControllers` 函数可以看出，是通过遍历的方式启动每一个 controller 的。其中传递了上下文参数 `ControllerCtx` ，它是每个控制器的上下文配置项。

```
// 启动所有控制器
// StartControllers starts a set of controllers with a specified ControllerContext
func StartControllers(ctx context.Context, controllerCtx ControllerContext, startSATokenController InitFunc, controllers map[string]InitFunc,
    unsecuredMux *mux.PathRecorderMux, healthzHandler *controllerhealthz.MutableHealthzHandler) error {

    // 遍历控制器
    for controllerName, initFn := range controllers {
        if !controllerCtx.IsControllerEnabled(controllerName) {
            logger.Info("Warning: controller is disabled", "controller", controllerName)
            continue
        }

                               time.Sleep(wait.Jitter(controllerCtx.ComponentConfig.Generic.ControllerStartInterval.Duration, ControllerStartJitter))

        logger.V(1).Info("Starting controller", "controller", controllerName)

        // 执行控制器，传递 controllerCtx 上下文配置
        ctrl, started, err := initFn(klog.NewContext(ctx, klog.LoggerWithName(logger, controllerName)), controllerCtx)

        // 控制器健康检测
        check := controllerhealthz.NamedPingChecker(controllerName)
        if ctrl != nil {
            // 1. 检查控制器是否支持调试 debugHandler
            if debuggable, ok := ctrl.(controller.Debuggable); ok && unsecuredMux != nil {
                if debugHandler := debuggable.DebuggingHandler(); debugHandler != nil {
                    basePath := "/debug/controllers/" + controllerName
                    unsecuredMux.UnlistedHandle(basePath, http.StripPrefix(basePath, debugHandler))
                    unsecuredMux.UnlistedHandlePrefix(basePath+"/", http.StripPrefix(basePath, debugHandler))
                }
            }

            // 2. 判断控制器是否包含一个 healthz endpoint，优先级高于上面的ping check
            if healthCheckable, ok := ctrl.(controller.HealthCheckable); ok {
                if realCheck := healthCheckable.HealthChecker(); realCheck != nil {
                    check = controllerhealthz.NamedHealthChecker(controllerName, realCheck)
                }
            }
        }
        controllerChecks = append(controllerChecks, check)

        logger.Info("Started controller", "controller", controllerName)
    }
    ...
}
```

这里我们以 `startDeploymentController()` 为例，看看 `deploymentController` 是如何启动的。

```
// cmd/kube-controller-manager/app/apps.go#L76
func startDeploymentController(ctx context.Context, controllerContext ControllerContext) (controller.Interface, bool, error) {
  // 1. 创建一个 DeploymentController.
    dc, err := deployment.NewDeploymentController(
        ctx,
        controllerContext.InformerFactory.Apps().V1().Deployments(),
        controllerContext.InformerFactory.Apps().V1().ReplicaSets(),
        controllerContext.InformerFactory.Core().V1().Pods(),
        controllerContext.ClientBuilder.ClientOrDie("deployment-controller"),
    )
    if err != nil {
        return nil, true, fmt.Errorf("error creating Deployment controller: %v", err)
    }

  // 2. 在一个goroutine中dc.Run()
    go dc.Run(ctx, int(controllerContext.ComponentConfig.DeploymentController.ConcurrentDeploymentSyncs))

    return nil, true, nil
}
```

这里调用了 `deployment.NewDeploymentController()` 函数，其中一些重要的参数全是通过我们上面说的 `controllerCtx` 来实现的。

最后在一个goroutine里执行 `dc.Run((ctx context.Context, workers int)`，这里的 `workers` 是指线程数，`deploymentController` 实现源码在 `pkg/controller/deployment/deployment_controller.go` ，就是一相标准的控制器实现，这个我们在上一节[《kubernetes 之 client-go 之 informer 工作原理源码解析》][2]中已经介绍过了，实现逻辑完全一模一样，这里不再做介绍。

# 总结 

可以看到对于 kube-controller-manager 来讲，其实逻辑还是很简单的，就是将内置所有controller 调用对应的启动函数就可以了。

对于每个controller 整体实现逻辑基本与上一篇文章介绍的步骤差不多，无非每个控制器实现的功能不一样而已。对于常见的控制器实现源码有时间的话，学习一下也不错。

# 参考资料 

 * [https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-controller-manager/](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-controller-manager/)
 * [https://blog.haohtml.com/archives/33163](https://blog.haohtml.com/archives/33163)

 [1]: https://github.com/kubernetes/kubernetes/blob/v1.27.3/cmd/kube-controller-manager/app/controllermanager.go
 [2]: https://blog.haohtml.com/archives/32179