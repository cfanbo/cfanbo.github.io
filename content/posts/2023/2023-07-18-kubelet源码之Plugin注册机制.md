---
title: kubelet 源码之 Plugin注册机制
author: admin
type: post
date: 2023-07-18T05:13:37+00:00
url: /archives/34275
categories:
 - 程序开发
tags:
 - k8s

---
上一篇[《Kubelet 服务引导流程》][1]我们讲了kubelet的大概引导流程, 本节我们看一下 `Plugins` 这一块的实现源码。

version: v1.27.3

# 插件模块入口 

入口文件 `/pkg/kubelet/kubelet.go`中的 `NewMainKubelet()` 函数，

```go
func NewMainKubelet(kubeCfg *kubeletconfiginternal.KubeletConfiguration,...) (*Kubelet, error) {
    ...
    // 插件管理器 /pkg/kubelet/kubelet.go#L811-L814
    klet.pluginManager = pluginmanager.NewPluginManager(
        klet.getPluginsRegistrationDir(), /* sockDir */
        kubeDeps.Recorder,
    )
    ...

}
```

这里第一个参数 `klet.getPluginsRegistrationDir()` 是返回 `plugins` 所在目录，默认位于`kubelet`目录下的 `plugins_registry` 目录，因此完整的路径为 `/var/lib/kubelet/plugins_registry`。（这些sock文件是由谁创建的呢？）同时将sock放在子目录里。

```go
const(
    DefaultKubeletPluginsRegistrationDirName = "plugins_registry"
)

func (kl *Kubelet) getPluginsRegistrationDir() string {
    return filepath.Join(kl.getRootDir(), config.DefaultKubeletPluginsRegistrationDirName)
}
```

第二个参数 `kubeDeps.Recorder` 是一个事件记录器 `EventRecorder` ，这里不需要关心。

我们看一下函数 `NewPluginManager()` 的实现。

```go
// pkg/kubelet/pluginmanager/plugin_manager.go#L54-L80
func NewPluginManager(
    sockDir string,
    recorder record.EventRecorder) PluginManager {
    asw := cache.NewActualStateOfWorld()
    dsw := cache.NewDesiredStateOfWorld()
    reconciler := reconciler.NewReconciler(
        operationexecutor.NewOperationExecutor(
            operationexecutor.NewOperationGenerator(
                recorder,
            ),
        ),
        loopSleepDuration,
        dsw,
        asw,
    )

    pm := &pluginManager{
        desiredStateOfWorldPopulator: pluginwatcher.NewWatcher(
            sockDir,
            dsw,
        ),
        reconciler:          reconciler,
        desiredStateOfWorld: dsw,
        actualStateOfWorld:  asw,
    }
    return pm
}
```

这里 `asw` 表示插件实际状态， `dsw` 表示插件期望状态，而 `reconciler` 就是用来 `reconciler` 这里的期望状态与实际状态的。

# asw实际状态 

查看函数 `cache.NewActualStateOfWorld()`

```go
// pkg/kubelet/pluginmanager/cache/actual_state_of_world.go
type ActualStateOfWorld interface {
    GetRegisteredPlugins() []PluginInfo
    AddPlugin(pluginInfo PluginInfo) error
    RemovePlugin(socketPath string)
    PluginExistsWithCorrectTimestamp(pluginInfo PluginInfo) bool
}

// NewActualStateOfWorld returns a new instance of ActualStateOfWorld
func NewActualStateOfWorld() ActualStateOfWorld {
    return &actualStateOfWorld{
        socketFileToInfo: make(map[string]PluginInfo),
    }
}
```

这里asw实际状态实现了接口 `ActualStateOfWorld`, 一共四个方法：

 1. `GetRegisteredPlugins()` 返回在当前实际状态注册的所有插件
 2. `AddPlugin()` 注册新插件到实际状态
 3. `RemovePlugin()` 从实际状态中根据插件的 `socketPath` 移除一个插件
 4. `PluginExistsWithCorrectTimestamp()` 判断一个插件是否在实际状态中注册且时间戳 `timestamp` 一致，这里要同时满足两个条件

这里我们看一下插件信息 `PluginInfo` 数据结构

```go
// pkg/kubelet/pluginmanager/cache/actual_state_of_world.go#L78
// PluginInfo holds information of a plugin
type PluginInfo struct {
    SocketPath string
    Timestamp  time.Time
    Handler    PluginHandler
    Name       string
}

// pkg/kubelet/pluginmanager/cache/types.go
// 插件处理器实现接口
type PluginHandler interface {
    ValidatePlugin(pluginName string, endpoint string, versions []string) error
    RegisterPlugin(pluginName string, endpoint string, versions []string) error
    DeRegisterPlugin(pluginName string)
}
```

对于插件的注册在内部实现是保证在一个map中的，其中`key`是 `PluginInfo.SocketPath` ,而 `value` 是 `PluginInfo` 结构体。

对于每一个插件 `PluginInfo.Handler` 要实现一个 `PluginHandler`接口，这里共三个方法。

 1. `ValidatePlugin` 检查插件信息是否有误，如版本号不支持
 2. `RegisterPlugin` 注册插件
 3. `DeRegisterPlugin` 一旦 `pluginwatcher`观察到套接字已被删除，就会调用 `DeRegisterPlugin` 方法

这三个方法的执行顺序

```
// The PluginHandler follows the simple following state machine:
//
//                           +--------------------------------------+
//                           |            ReRegistration            |
//                           | Socket created with same plugin name |
//                           |                                      |
//                           |                                      |
//      Socket Created       v                                      +        Socket Deleted
//    +------------------> Validate +---------------------------> Register +------------------> DeRegister
//                           +                                      +                              +
//                           |                                      |                              |
//                           | Error                                | Error                        |
//                           |                                      |                              |
//                           v                                      v                              v
//                          Out                                    Out                            Out
//
```

# dst期望状态 

查看函数 `cache.NewDesiredStateOfWorld()`

```go
// pkg/kubelet/pluginmanager/cache/desired_state_of_world.go
type DesiredStateOfWorld interface {
    AddOrUpdatePlugin(socketPath string) error
    RemovePlugin(socketPath string)
    GetPluginsToRegister() []PluginInfo
    PluginExists(socketPath string) bool
}

// NewDesiredStateOfWorld returns a new instance of DesiredStateOfWorld.
func NewDesiredStateOfWorld() DesiredStateOfWorld {
    return &desiredStateOfWorld{
        socketFileToInfo: make(map[string]PluginInfo),
    }
}
```

这里实现了接口 `DesiredStateOfWorld`，共四个方法:

 1. `AddOrUpdatePlugin` 添加一个插件到期望状态，如果插件不存在则直接添加；如果插件已存在，则更新插件的时间戳 `PluginInfo.Timestamp` 字段
 2. `RemovePlugin` 从期望状态中删除一个插件
 3. `GetPluginsToRegister` 从期望状态中返回插件列表
 4. `PluginExists` 检查插件是否在期望状态中存在

对于`dsw`插件在内部实现上同`asw` 一样，也是通过一个`map` 数据结构来实现的。

# reconciler 协调器 

上面介绍了插件的 `期望状态` 与 `实际状态`，剩下的就是对这两种状态的 `reconciler` 操作了。

```go
func NewPluginManager(sockDir string,...) PluginManager {
  ...
  reconciler := reconciler.NewReconciler(
        operationexecutor.NewOperationExecutor(
            operationexecutor.NewOperationGenerator(
                recorder,
            ),
        ),
        loopSleepDuration,
        dsw,
        asw,
    )
  ...
}
```

函数 `NewReconciler()` 源码

```go
// pkg/kubelet/pluginmanager/reconciler/reconciler.go
type Reconciler interface {
    Run(stopCh <-chan struct{})
    AddHandler(pluginType string, pluginHandler cache.PluginHandler)
}

type reconciler struct {
    operationExecutor   operationexecutor.OperationExecutor
    loopSleepDuration   time.Duration
    desiredStateOfWorld cache.DesiredStateOfWorld
    actualStateOfWorld  cache.ActualStateOfWorld
    handlers            map[string]cache.PluginHandler
    sync.RWMutex
}

func NewReconciler(
    operationExecutor operationexecutor.OperationExecutor,
    loopSleepDuration time.Duration,
    desiredStateOfWorld cache.DesiredStateOfWorld,
    actualStateOfWorld cache.ActualStateOfWorld) Reconciler {
    return &reconciler{
        operationExecutor:   operationExecutor,
        loopSleepDuration:   loopSleepDuration,
        desiredStateOfWorld: desiredStateOfWorld,
        actualStateOfWorld:  actualStateOfWorld,
        handlers:            make(map[string]cache.PluginHandler),
    }
}
```

函数共有四个参数：

 1. `operationExecutor`: operationExecutor-用于安全地触发注册/注销操作（防止在同一套接字路径上触发多个操作
 2. `loopSleepDuration` 协调器循环在连续执行之间休眠的时间
 3. `desiredStateOfWorld` 期望状态
 4. `actualStateOfWorld` 实现状态

其实现了 `Reconciler` 接口，此接口共两个方法：

 1. `Run` 启动 `pluginManager` 并异步执行Loop循环
 2. `AddHandler(pluginType string, pluginHandler cache.PluginHandler)` 添加指定类型的插件到`实际状态`，以便传递给期望状态，以便在 `registration/deregistration` 期间调用。

我们看一下 `reconciler` 对这个接口的真正实现

```go
// pkg/kubelet/pluginmanager/reconciler/reconciler.go

// 共三个参数，第一个参数是一个函数
func (rc *reconciler) Run(stopCh <-chan struct{}) {
    wait.Until(func() {
        rc.reconcile()
    },
        rc.loopSleepDuration,
        stopCh)
}

// 注册插件类型
func (rc *reconciler) AddHandler(pluginType string, pluginHandler cache.PluginHandler) {
    rc.Lock()
    defer rc.Unlock()

    rc.handlers[pluginType] = pluginHandler
}
```

我看一下 Run() 函数里的 `rc.reconcile()`

```go
func (rc *reconciler) reconcile() {
    // Unregisterations are triggered before registrations

  // 取消插件注册
    // Ensure plugins that should be unregistered are unregistered.
    for _, registeredPlugin := range rc.actualStateOfWorld.GetRegisteredPlugins() {
        unregisterPlugin := false
    // 实际状态中的插件不在期望状态中存在，则取消插件注册
        if !rc.desiredStateOfWorld.PluginExists(registeredPlugin.SocketPath) {
            unregisterPlugin = true
        } else {
      // 如果插件在两种状态中都存在，但 timestamp 不一样，则取消插件注册
            // We also need to unregister the plugins that exist in both actual state of world
            // and desired state of world cache, but the timestamps don't match.
            // Iterate through desired state of world plugins and see if there's any plugin
            // with the same socket path but different timestamp.
            for _, dswPlugin := range rc.desiredStateOfWorld.GetPluginsToRegister() {
                if dswPlugin.SocketPath == registeredPlugin.SocketPath && dswPlugin.Timestamp != registeredPlugin.Timestamp {
                    klog.V(5).InfoS("An updated version of plugin has been found, unregistering the plugin first before reregistering", "plugin", registeredPlugin)
                    unregisterPlugin = true
                    break
                }
            }
        }

    // 取消插件注册 （operationExecutor）
        if unregisterPlugin {
            klog.V(5).InfoS("Starting operationExecutor.UnregisterPlugin", "plugin", registeredPlugin)
            err := rc.operationExecutor.UnregisterPlugin(registeredPlugin, rc.actualStateOfWorld)
            if err != nil &&
                !goroutinemap.IsAlreadyExists(err) &&
                !exponentialbackoff.IsExponentialBackoff(err) {
                // Ignore goroutinemap.IsAlreadyExists and exponentialbackoff.IsExponentialBackoff errors, they are expected.
                // Log all other errors.
                klog.ErrorS(err, "OperationExecutor.UnregisterPlugin failed", "plugin", registeredPlugin)
            }
            if err == nil {
                klog.V(1).InfoS("OperationExecutor.UnregisterPlugin started", "plugin", registeredPlugin)
            }
        }
    }

  // 注册插件
    // Ensure plugins that should be registered are registered
    for _, pluginToRegister := range rc.desiredStateOfWorld.GetPluginsToRegister() {
    // 如果期望状态中的插件不在实际状态中存在，则直接注册这个插件
        if !rc.actualStateOfWorld.PluginExistsWithCorrectTimestamp(pluginToRegister) {
            klog.V(5).InfoS("Starting operationExecutor.RegisterPlugin", "plugin", pluginToRegister)
            err := rc.operationExecutor.RegisterPlugin(pluginToRegister.SocketPath, pluginToRegister.Timestamp, rc.getHandlers(), rc.actualStateOfWorld)
            if err != nil &&
                !goroutinemap.IsAlreadyExists(err) &&
                !exponentialbackoff.IsExponentialBackoff(err) {
                // Ignore goroutinemap.IsAlreadyExists and exponentialbackoff.IsExponentialBackoff errors, they are expected.
                klog.ErrorS(err, "OperationExecutor.RegisterPlugin failed", "plugin", pluginToRegister)
            }
            if err == nil {
                klog.V(1).InfoS("OperationExecutor.RegisterPlugin started", "plugin", pluginToRegister)
            }
        }
    }
}
```

这里插件的注册与取消注册是通过 `rc.operationExecutor` （ 操作执行器）来完成的，它是在上在函数 `NewPluginManager()` 中调用函数来实现的。

```go
    operationexecutor.NewOperationExecutor(
        operationexecutor.NewOperationGenerator(
            recorder,
        ),
    )
```

函数实现为

```go
// pkg/kubelet/pluginmanager/operationexecutor/operation_executor.go
// NewOperationExecutor() 实现接口
type OperationExecutor interface {
    // 注册插件
    RegisterPlugin(socketPath string, timestamp time.Time, pluginHandlers map[string]cache.PluginHandler, actualStateOfWorld ActualStateOfWorldUpdater) error

    // 取消注册插件
    UnregisterPlugin(pluginInfo cache.PluginInfo, actualStateOfWorld ActualStateOfWorldUpdater) error
}

type ActualStateOfWorldUpdater interface {
    AddPlugin(pluginInfo cache.PluginInfo) error
    RemovePlugin(socketPath string)
}

type operationExecutor struct {
    pendingOperations goroutinemap.GoRoutineMap
    operationGenerator OperationGenerator
}

// 创建函数
func NewOperationExecutor(
    operationGenerator OperationGenerator) OperationExecutor {

    return &operationExecutor{
        pendingOperations:  goroutinemap.NewGoRoutineMap(true /* exponentialBackOffOnError */),
        operationGenerator: operationGenerator,
    }
}
```

对接口的实现

```go
// pkg/kubelet/pluginmanager/operationexecutor/operation_executor.go

// 执行器注册插件
func (oe *operationExecutor) RegisterPlugin(
    socketPath string,
    timestamp time.Time,
    pluginHandlers map[string]cache.PluginHandler,
    actualStateOfWorld ActualStateOfWorldUpdater) error {

    // 变量函数
    generatedOperation :=
        oe.operationGenerator.GenerateRegisterPluginFunc(socketPath, timestamp, pluginHandlers, actualStateOfWorld)

    return oe.pendingOperations.Run(
        socketPath, generatedOperation)
}

// 执行器取消注册插件
func (oe *operationExecutor) UnregisterPlugin(
    pluginInfo cache.PluginInfo,
    actualStateOfWorld ActualStateOfWorldUpdater) error {

    // 变量函数
    generatedOperation :=
        oe.operationGenerator.GenerateUnregisterPluginFunc(pluginInfo, actualStateOfWorld)

    return oe.pendingOperations.Run(
        pluginInfo.SocketPath, generatedOperation)
}
```

可以看到对于插件的注册和取消注册统一调用的是 `oe.pendingOperations.Run()` 函数。

这里 `oe.pendingOperations` 是一个 `GoRoutineMap`接口

```go
// pkg/util/goroutinemap/goroutinemap.go
type GoRoutineMap interface {
        // 第二个参数是函数
    Run(operationName string, operationFunc func() error) error
    Wait()
    WaitForCompletion()
    IsOperationPending(operationName string) bool
}
```

实现

```go
type goRoutineMap struct {
    operations                map[string]operation
    exponentialBackOffOnError bool
    cond                      *sync.Cond
    lock                      sync.RWMutex
}
type operation struct {
    operationPending bool
    expBackoff       exponentialbackoff.ExponentialBackoff
}

// 执行器接口实现
func (grm *goRoutineMap) Run(
    operationName string,
    operationFunc func() error) error {
    grm.lock.Lock()
    defer grm.lock.Unlock()

    existingOp, exists := grm.operations[operationName]
    if exists {
        // Operation with name exists
        if existingOp.operationPending {
            return NewAlreadyExistsError(operationName)
        }

        if err := existingOp.expBackoff.SafeToRetry(operationName); err != nil {
            return err
        }
    }

  // 注册操作
    grm.operations[operationName] = operation{
        operationPending: true,
        expBackoff:       existingOp.expBackoff,
    }

    // 在一个单独的 goroutine 执行回调
    go func() (err error) {
        // Handle unhandled panics (very unlikely)
        defer k8sRuntime.HandleCrash()
        // Handle completion of and error, if any, from operationFunc()
        defer grm.operationComplete(operationName, &err)
        // Handle panic, if any, from operationFunc()
        defer k8sRuntime.RecoverFromPanic(&err)

        // 执行回调
        return operationFunc()
    }()

    return nil
}
```

这里对于回调函数 `operationFunc()` 的实现接口 `OperationGenerator`

```go
// pkg/kubelet/pluginmanager/operationexecutor/operation_generator.go
type OperationGenerator interface {
    GenerateRegisterPluginFunc(socketPath string, timestamp time.Time, pluginHandlers map[string]cache.PluginHandler, actualStateOfWorldUpdater ActualStateOfWorldUpdater) func() error
    GenerateUnregisterPluginFunc(pluginInfo cache.PluginInfo, actualStateOfWorldUpdater ActualStateOfWorldUpdater) func() error
}
```

注册插件实现

```go
// pkg/kubelet/pluginmanager/operationexecutor/operation_generator.go

// 注册插件回调函数
func (og *operationGenerator) GenerateRegisterPluginFunc(
    socketPath string,
    timestamp time.Time,
    pluginHandlers map[string]cache.PluginHandler,
    actualStateOfWorldUpdater ActualStateOfWorldUpdater) func() error {

    registerPluginFunc := func() error {
        // 1. 连接gRPCServer
        client, conn, err := dial(socketPath, dialTimeoutDuration)
        if err != nil {
            return fmt.Errorf("RegisterPlugin error -- dial failed at socket %s, err: %v", socketPath, err)
        }
        defer conn.Close()

        ctx, cancel := context.WithTimeout(context.Background(), time.Second)
        defer cancel()

        // 2. 获取插件信息 PluginInfo
        infoResp, err := client.GetInfo(ctx, ®isterapi.InfoRequest{})
        if err != nil {
            return fmt.Errorf("RegisterPlugin error -- failed to get plugin info using RPC GetInfo at socket %s, err: %v", socketPath, err)
        }

        handler, ok := pluginHandlers[infoResp.Type]
        if !ok {
            if err := og.notifyPlugin(client, false, fmt.Sprintf("RegisterPlugin error -- no handler registered for plugin type: %s at socket %s", infoResp.Type, socketPath)); err != nil {
                return fmt.Errorf("RegisterPlugin error -- failed to send error at socket %s, err: %v", socketPath, err)
            }
            return fmt.Errorf("RegisterPlugin error -- no handler registered for plugin type: %s at socket %s", infoResp.Type, socketPath)
        }

        // 3. 校验Plugin （检查版本是否有效）这里 infoResp.Endpoint 的值是 /var/lib/kubelet/plugins/xxxxx
        if infoResp.Endpoint == "" {
            infoResp.Endpoint = socketPath
        }
        if err := handler.ValidatePlugin(infoResp.Name, infoResp.Endpoint, infoResp.SupportedVersions); err != nil {
            if err = og.notifyPlugin(client, false, fmt.Sprintf("RegisterPlugin error -- plugin validation failed with err: %v", err)); err != nil {
                return fmt.Errorf("RegisterPlugin error -- failed to send error at socket %s, err: %v", socketPath, err)
            }
            return fmt.Errorf("RegisterPlugin error -- pluginHandler.ValidatePluginFunc failed")
        }
        // 4. 添加Plugin到实际状态,  先更新cache
        // We add the plugin to the actual state of world cache before calling a plugin consumer's Register handle
        // so that if we receive a delete event during Register Plugin, we can process it as a DeRegister call.
        err = actualStateOfWorldUpdater.AddPlugin(cache.PluginInfo{
            SocketPath: socketPath,
            Timestamp:  timestamp,
            Handler:    handler,
            Name:       infoResp.Name,
        })
        if err != nil {
            klog.ErrorS(err, "RegisterPlugin error -- failed to add plugin", "path", socketPath)
        }
        // 5. 注册插件(将csi的nodeID写到node的annotation中，并且创建/更新csinode)
        if err := handler.RegisterPlugin(infoResp.Name, infoResp.Endpoint, infoResp.SupportedVersions); err != nil {
            return og.notifyPlugin(client, false, fmt.Sprintf("RegisterPlugin error -- plugin registration failed with err: %v", err))
        }

        // 6. 发送在kubelet注册插件成功
        // Notify is called after register to guarantee that even if notify throws an error Register will always be called after validate
        if err := og.notifyPlugin(client, true, ""); err != nil {
            return fmt.Errorf("RegisterPlugin error -- failed to send registration status at socket %s, err: %v", socketPath, err)
        }
        return nil
    }
    return registerPluginFunc
}
```

注册步骤：

 1. 通过 `sock` 文件与Pod建立`gRPC` 连接（本文暂不介绍 grpcServer 组件）
 2. 获取插件信息 PluginInfo
 3. 验证插件是否有效，检检查版本是否一致。
 4. 添加 Plugin到实际状态cache
 5. 注册插件(将csi的nodeID写到node的annotation中，并且创建/更新csinode)
 6. 发送插件注册成功消息给 grpcServer![](https://blogstatic.haohtml.com/uploads/2023/07/d2b5ca33bd970f64a6301fa75ae2eb22.png)

取消注册插件

```go
// pkg/kubelet/pluginmanager/operationexecutor/operation_generator.go

func (og *operationGenerator) GenerateUnregisterPluginFunc(
    pluginInfo cache.PluginInfo,
    actualStateOfWorldUpdater ActualStateOfWorldUpdater) func() error {

    unregisterPluginFunc := func() error {
        if pluginInfo.Handler == nil {
            return fmt.Errorf("UnregisterPlugin error -- failed to get plugin handler for %s", pluginInfo.SocketPath)
        }

        // 1. 先从实际状态cache中删除插件
        // We remove the plugin to the actual state of world cache before calling a plugin consumer's Unregister handle
        // so that if we receive a register event during Register Plugin, we can process it as a Register call.
        actualStateOfWorldUpdater.RemovePlugin(pluginInfo.SocketPath)

        // 2. 取消注册
        pluginInfo.Handler.DeRegisterPlugin(pluginInfo.Name)

        klog.V(4).InfoS("DeRegisterPlugin called", "pluginName", pluginInfo.Name, "pluginHandler", pluginInfo.Handler)
        return nil
    }
    return unregisterPluginFunc
}
```

取消步骤：

 1. 先从实际状态cache中删除插件信息
 2. 取消注册插件，当 _pluginwatcher_ 观察到sock被删除时，将调用这个函数

于是现在有两个问题：一个是 pluginwatcher 是什么时候创建的？另一个是 sock 文件什么时候被删除？

# pluginWatcher 

对于`pluginwatcher` 它是在 `NewPluginManager()` 创建的

```go
func NewPluginManager() PluginManager {
    ...
    pm := &pluginManager{
        desiredStateOfWorldPopulator: pluginwatcher.NewWatcher(
            sockDir,
            dsw,
        ),
        reconciler:          reconciler,
        desiredStateOfWorld: dsw,
        actualStateOfWorld:  asw,
    }
    return pm
}
```

```go
// pkg/kubelet/pluginmanager/pluginwatcher/plugin_watcher.go
func NewWatcher(sockDir string, desiredStateOfWorld cache.DesiredStateOfWorld) *Watcher {
    return &Watcher{
        path:                sockDir,
        fs:                  &utilfs.DefaultFs{},
        desiredStateOfWorld: desiredStateOfWorld,
    }
}

func (w *Watcher) Start(stopCh <-chan struct{}) error {
    klog.V(2).InfoS("Plugin Watcher Start", "path", w.path)

    // Creating the directory to be watched if it doesn't exist yet,
    // and walks through the directory to discover the existing plugins.
    if err := w.init(); err != nil {
        return err
    }

    // 1. 创建 fsnotify 对象
    fsWatcher, err := fsnotify.NewWatcher()
    if err != nil {
        return fmt.Errorf("failed to start plugin fsWatcher, err: %v", err)
    }
    w.fsWatcher = fsWatcher

    // 2. 监控sock, 如果是目录的话，则添加到监控清单
    // Traverse plugin dir and add filesystem watchers before starting the plugin processing goroutine.
    if err := w.traversePluginDir(w.path); err != nil {
        klog.ErrorS(err, "Failed to traverse plugin socket path", "path", w.path)
    }

    // 3. 事件处理
    go func(fsWatcher *fsnotify.Watcher) {
        for {
            select {
            case event := <-fsWatcher.Events:
                //TODO: Handle errors by taking corrective measures
                if event.Has(fsnotify.Create) {
                    err := w.handleCreateEvent(event)
                    if err != nil {
                        klog.ErrorS(err, "Error when handling create event", "event", event)
                    }
                } else if event.Has(fsnotify.Remove) {
                    w.handleDeleteEvent(event)
                }
                continue
            case err := <-fsWatcher.Errors:
                if err != nil {
                    klog.ErrorS(err, "FsWatcher received error")
                }
                continue
            case <-stopCh:
                w.fsWatcher.Close()
                return
            }
        }
    }(fsWatcher)

    return nil
}
```

**插件注册**

这里通过 `fsnotify` 来监控插件 sock 文件的变化情况，如果这个路径是一个目录的话，则将这个目录添加到监控列表中。如果监听到文件的创建，则执行 `w.handleCreateEvent()`函数。

```go
// pkg/kubelet/pluginmanager/pluginwatcher/plugin_watcher.go

func (w *Watcher) handleCreateEvent(event fsnotify.Event) error {
    klog.V(6).InfoS("Handling create event", "event", event)

    fi, err := getStat(event)
    if err != nil {
        return fmt.Errorf("stat file %s failed: %v", event.Name, err)
    }

    if strings.HasPrefix(fi.Name(), ".") {
        klog.V(5).InfoS("Ignoring file (starts with '.')", "path", fi.Name())
        return nil
    }

    // 如果是一个 sock 文件，则调用   w.handlePluginRegistration(event.Name)
    if !fi.IsDir() {
        isSocket, err := util.IsUnixDomainSocket(util.NormalizePath(event.Name))
        if err != nil {
            return fmt.Errorf("failed to determine if file: %s is a unix domain socket: %v", event.Name, err)
        }
        if !isSocket {
            klog.V(5).InfoS("Ignoring non socket file", "path", fi.Name())
            return nil
        }

        return w.handlePluginRegistration(event.Name)
    }

    // 否则调用 w.traversePluginDir(event.Name)
    return w.traversePluginDir(event.Name)
}
```

如果变化的是一个sock文件，则直接调用 `w.handlePluginRegistration(event.Name)` 否则调用 `w.traversePluginDir(event.Name)` 遍历整个目录，针对目录里面的 sock 文件 再次调用 `w.handleCreateEvent()`。

现在看一下 `Wathcer` 是如何如何注册插件的

```go
func (w *Watcher) handlePluginRegistration(socketPath string) error {
    socketPath = getSocketPath(socketPath)

    // 将观察到的插件注册到期望状态 dsw 中
    err := w.desiredStateOfWorld.AddOrUpdatePlugin(socketPath)
    if err != nil {
        return fmt.Errorf("error adding socket path %s or updating timestamp to desired state cache: %v", socketPath, err)
    }
    return nil
}
```

可以看到对于 pluginWatcher 来讲，当发现有插件sock的时候，会将这个插件注册到`dsw`中

```go
// pkg/kubelet/pluginmanager/cache/desired_state_of_world.go#L138-L141
func (dsw *desiredStateOfWorld) AddOrUpdatePlugin(socketPath string) error {
    // 注册插件，此时缺少插件 PluginInfo.Name 和  PluginInfo.Handler
    dsw.socketFileToInfo[socketPath] = PluginInfo{
        SocketPath: socketPath,
        Timestamp:  time.Now(),
    }
    return nil
}
```

可以看到在这里注册插件信息只有部分信息，重要的 `PluginInfo.Handler` 目前为`nil` ，也就是说插件暂时不可使用。

**插件删除**

当 `pluginWatch` 监控到删除操作时，将调用 `w.handleDeleteEvent(event)` 处理

```go
func (w *Watcher) handleDeleteEvent(event fsnotify.Event) {
    socketPath := event.Name
    w.desiredStateOfWorld.RemovePlugin(socketPath)
}
```

可以看到这里是将插件从 dsw 中删除， 真正执行的代码为

```go
// pkg/kubelet/pluginmanager/cache/desired_state_of_world.go#L145-L150
func (dsw *desiredStateOfWorld) RemovePlugin(socketPath string) {
    dsw.Lock()
    defer dsw.Unlock()

    delete(dsw.socketFileToInfo, socketPath)
}
```

总结:

对于 pluginWatcher 来讲，是通过监控插件sock或目录来实现插件的注册与取消注册。如果发现新的插件sock文件，则将其注册到 `dsw` 状态中，反之从`dsw`中删除。

**pluginManager 启动**

对于PluginManager 服务启动的完整调用链路为

`func (kl *Kubelet) Run()` ->
`go wait.Until(kl.updateRuntimeUp, 5*time.Second, wait.NeverStop)` ->
`kl.oneTimeInitializer.Do(kl.initializeRuntimeDependentModules)` ->
`go kl.pluginManager.Run(kl.sourcesReady, wait.NeverStop)`

其中在 `initializeRuntimeDependentModules`这一步，先是注册了一些不同类型的插件，然后才正式启动服务。

```go
func (kl *Kubelet) initializeRuntimeDependentModules() {
    // 1. CSI Driver 插件回调函数， pkg/volume/csi/csi_plugin.go
  // Adding Registration Callback function for CSI Driver
    kl.pluginManager.AddHandler(pluginwatcherapi.CSIPlugin, plugincache.PluginHandler(csi.PluginHandler))

    // 2. ORA 插件注册回调函数
    // Adding Registration Callback function for DRA Plugin
    if utilfeature.DefaultFeatureGate.Enabled(features.DynamicResourceAllocation) {
        kl.pluginManager.AddHandler(pluginwatcherapi.DRAPlugin, plugincache.PluginHandler(draplugin.NewRegistrationHandler()))
    }

    // 3. Device Manager 注册回调函数
    // Adding Registration Callback function for Device Manager
    kl.pluginManager.AddHandler(pluginwatcherapi.DevicePlugin, kl.containerManager.GetPluginRegistrationHandler())

    // 4. Start the plugin manager
    klog.V(4).InfoS("Starting plugin manager")
    go kl.pluginManager.Run(kl.sourcesReady, wait.NeverStop)

  ...
}
```

源码实现内置注册了三类插件，最后才启动服务。 其中一个为 `CSI Driver`，它是k8s中一个很重要的一个知识点，后续可能单独来看一下它的实现。

# 总结 

对于 `plugin` 的维护是由 `pluginManager` 通过 `fsnotify` 对 插件 sock 进行监控来实现的，当发现新插件时，会将其在期望状态 `dsw` 注册，反之一样。

对于期望状态 `dsw` 和 实际状态 `asw` 会定期 `rc.loopSleepDuration`(默认1秒) 通过 `reconciler` 同步两者，保持插件信息一致。

这里并未介绍到对于插件sock的产生逻辑，这一块可能再花一些时间了解一下。

# 参考资料 

 * [https://github.com/kubernetes/kubernetes/blob/v1.27.3/pkg/kubelet/kubelet.go](https://github.com/kubernetes/kubernetes/blob/v1.27.3/pkg/kubelet/kubelet.go)
 * [https://www.cnblogs.com/zhangmingcheng/p/17107702.html](https://www.cnblogs.com/zhangmingcheng/p/17107702.html)

 [1]: https://blog.haohtml.com/archives/33188