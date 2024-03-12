---
title: 如何在一个pod中创建容器
date: 2023-12-05T11:07:50+08:00
draft: true
---



![architecture](https://github.com/containerd/containerd/raw/main/docs/cri/architecture.png)



上图是来自官方提供的[架构图](https://github.com/containerd/containerd/blob/main/docs/cri/architecture.md)，它描述了一个pod的整个创建流程，我们这本节重点关注的就是最后一步，即如何在一个 pod中创建一个容器。



创建一个容器

每次创建一个pod容器时，在 `kubelet` 组件中将执行 `start()` 函数

```go
func (m *kubeGenericRuntimeManager) SyncPod(ctx context.Context, pod *v1.Pod, podStatus *kubecontainer.PodStatus, pullSecrets []v1.Secret, backOff *flowcontrol.Backoff) (result kubecontainer.PodSyncResult) {
	
	...
	// Helper containing boilerplate common to starting all types of containers.
	// typeName is a description used to describe this type of container in log messages,
	// currently: "container", "init container" or "ephemeral container"
	// metricLabel is the label used to describe this type of container in monitoring metrics.
	// currently: "container", "init_container" or "ephemeral_container"
	start := func(ctx context.Context, typeName, metricLabel string, spec *startSpec) error {
		startContainerResult := kubecontainer.NewSyncResult(kubecontainer.StartContainer, spec.container.Name)
		result.AddSyncResult(startContainerResult)
		

		// 启用一个pod容器，下面对介绍它的实现
		if msg, err := m.startContainer(ctx, podSandboxID, podSandboxConfig, spec, pod, podStatus, pullSecrets, podIP, podIPs); err != nil {
			
			...
			return err
		}

		return nil
	}  
  
  ...
  
  
}
```

可以用 `start()` 函数实现 `container`, `init container` or `ephemeral container`  三类容器的创建。

typeName 表示容器类型； metricLabel 为容器可观测性指标名称，spec 表示容器信息，结构体声明

```go
https://github.com/kubernetes/kubernetes/blob/b5fd7e9f23c6ce8d3d3f325184359ce4ccb81d25/pkg/kubelet/kuberuntime/kuberuntime_container.go#L99-L105
type startSpec struct {
	container          *v1.Container
	ephemeralContainer *v1.EphemeralContainer
}
```

字段 `v1.Container` 就是核心容器结构体声明，而 `v1.EphemeralContainer` 的结构为

```go
type EphemeralContainer struct {
	EphemeralContainerCommon `json:",inline" protobuf:"bytes,1,req"`

	TargetContainerName string `json:"targetContainerName,omitempty" protobuf:"bytes,2,opt,name=targetContainerName"`
}

// EphemeralContainerCommon是 v1.Container 中所有字段的副本，这些字段将被内联到EphemicalContainer中。这种单独的类型允许从EphemeralContainer轻松转换为Container，并允许Ephemeral Container字段的单独文档。
type EphemeralContainerCommon struct {
 	... 
}
```

其中 `EphemeralContainerCommon` 结构体是对 `v1.Container` 的复制, 可以将其类型转换为 `v1.Container` , 也就是说 `EphemeralContainer` 其实是对 `v1.Container` 的一种封装而已，以上对这两类容器之间的关系。

对pod里容器的创建是以 [startContainer() ](https://github.com/kubernetes/kubernetes/blob/b5fd7e9f23c6ce8d3d3f325184359ce4ccb81d25/pkg/kubelet/kuberuntime/kuberuntime_container.go#L71-300) 函数开始的。

```go
// startContainer starts a container and returns a message indicates why it is failed on error.
// It starts the container through the following steps:
// * pull the image
// * create the container
// * start the container
// * run the post start lifecycle hooks (if applicable)
func (m *kubeGenericRuntimeManager) startContainer(ctx context.Context, podSandboxID string, podSandboxConfig *runtimeapi.PodSandboxConfig, spec *startSpec, pod *v1.Pod, podStatus *kubecontainer.PodStatus, pullSecrets []v1.Secret, podIP string, podIPs []string) (string, error) {
	container := spec.container
  
  
  // Step 1: pull the image.
	imageRef, msg, err := m.imagePuller.EnsureImageExists(ctx, pod, container, pullSecrets, podSandboxConfig)
  
  // Step 2: create the container.
	// For a new container, the RestartCount should be 0
  
  // 2.1 PreCreateContainer()
  err = m.internalLifecycle.PreCreateContainer(pod, container, containerConfig)
  
  // 2.2 调用 CreateContainer() 函数创建容器
  containerID, err := m.runtimeService.CreateContainer(ctx, podSandboxID, containerConfig, podSandboxConfig)
  
  // 2.3 PreStartContainer()
  err = m.internalLifecycle.PreStartContainer(pod, container, containerID)
	
  // Step 3: start the container.
	err = m.runtimeService.StartContainer(ctx, containerID)
  
  // Step 4: execute the post start hook.
	if container.Lifecycle != nil && container.Lifecycle.PostStart != nil {
    kubeContainerID := kubecontainer.ContainerID{
			Type: m.runtimeName,
			ID:   containerID,
		}
		msg, handlerErr := m.runner.Run(ctx, kubeContainerID, pod, container, container.Lifecycle.PostStart)
    if handlerErr != nil {
     	... 
    }
  }
  
}
```

创建一个pod容器流程大致分为四步：

1. 下载容器镜像
2. 创建容器，我们关注的重点。
3. 启动容器
4. 执行 post start 钩子函数，如果有的话

在创建容器时，需要对与容器准备或收尾工作相关的工作。

```go
// https://github.com/kubernetes/kubernetes/blob/33d8614c9cc5f8f4a048117a00677043e28c1704/pkg/kubelet/cm/internal_container_lifecycle.go
type InternalContainerLifecycle interface {
	PreCreateContainer(pod *v1.Pod, container *v1.Container, containerConfig *runtimeapi.ContainerConfig) error
  
	PreStartContainer(pod *v1.Pod, container *v1.Container, containerID string) error
  
	PostStopContainer(containerID string) error
}
```

比如在真正创建容器前，调用 `PreCreateContainer()` 做一些容器准备工作；在启动容器前，调用 `PreStartContainer()` 也一些初始化工作；除此之外，还可以在 停止容器后，调用 `PostStopContainer()` 做一些收尾工作。

上面我们重点关心的是 `m.runtimeService.CreateContainer()` 如何创建一个容器过程，这里 `runtimeService` 是一个运行时接口，对它的实现有多种，我们这里以 `containerd` 这个运行时为例。



> `runtimeService` 是通过 gRPC 来实现的与 containerd  通讯的，其在 `pkg/kubelet/cri/remote/remote_runtime.go#L304` 文件中可以找到 `CreateContainer()` 函数定义。
>
> 对应的 proto 文件 为 `staging/src/k8s.io/cri-api/pkg/apis/runtime/v1/api.proto `，请求体为 `CreateContainerRequest`， 
>
> ```go
> message CreateContainerRequest {
>     // ID of the PodSandbox in which the container should be created.
>     string pod_sandbox_id = 1;
>     // Config of the container.
>     ContainerConfig config = 2;
>     // Config of the PodSandbox. This is the same config that was passed
>     // to RunPodSandboxRequest to create the PodSandbox. It is passed again
>     // here just for easy reference. The PodSandboxConfig is immutable and
>     // remains the same throughout the lifetime of the pod.
>     PodSandboxConfig sandbox_config = 3;
> }
> ```
>
> 响应体为 `CreateContainerResponse`,
>
> ```go
> message CreateContainerResponse {
>     // ID of the created container.
>     string container_id = 1;
> }
> ```
>
> 客户端调用
>
> ```go
> // pkg/kubelet/cri/remote/remote_runtime.go
> func (r *remoteRuntimeService) createContainerV1(ctx context.Context, podSandBoxID string, config *runtimeapi.ContainerConfig, sandboxConfig *runtimeapi.PodSandboxConfig) (string, error) {
> 	resp, err := r.runtimeClient.CreateContainer(ctx, &runtimeapi.CreateContainerRequest{
> 		PodSandboxId:  podSandBoxID,
> 		Config:        config,
> 		SandboxConfig: sandboxConfig,
> 	})
> 	if err != nil {
> 		return "", err
> 	}
> 
> 	if resp.ContainerId == "" {
> 		errorMessage := fmt.Sprintf("ContainerId is not set for container %q", config.Metadata)
> 		err := errors.New(errorMessage)
> 		return "", err
> 	}
> 
> 	return resp.ContainerId, nil
> }
> ```

切记：对于 kubelet 组件 和 运行时 containerd 它们运行在同一台机器。

对于 containerd 运行时创建容器的实现位于 https://github.com/containerd/containerd/blob/32bf805e5703bc91387d047fa76625e915ac2b80/pkg/cri/server/container_create.go#L57

```go
// CreateContainer creates a new container in the given PodSandbox.
func (c *criService) CreateContainer(ctx context.Context, r *runtime.CreateContainerRequest) (_ *runtime.CreateContainerResponse, retErr error) {

  ...
  
	return &runtime.CreateContainerResponse{ContainerId: id}, nil
}
```

注意创建容器时，将根据 `PodSandbox` 的配置来实现

```go
	// sandboxService 是一个sandbox 相关的service
	controller, err := c.sandboxService.SandboxController(sandbox.Config, sandbox.RuntimeHandler)
	cstatus, err := controller.Status(ctx, sandbox.ID, false)

	// 容器镜像本地检查
	image, err := c.LocalResolve(config.GetImage().GetImage())

	// 创建容器目录
	// Create container root directory. 
	containerRootDir := c.getContainerRootDir(id)
	if err = c.os.MkdirAll(containerRootDir, 0755); err != nil {
		return nil, fmt.Errorf("failed to create container root directory %q: %w",
			containerRootDir, err)
	}
	var volumeMounts []*runtime.Mount
	if !c.config.IgnoreImageDefinedVolumes {
    // 收集image中指定的volumns,需要考虑当前容器运行的 platform 是Linux（对应 user namespace 中的uid 和gid）还是其它系统
		// Create container image volumes mounts.
		volumeMounts = c.volumeMounts(platform, containerRootDir, config, &image.ImageSpec.Config)
	} else if len(image.ImageSpec.Config.Volumes) != 0 {
		log.G(ctx).Debugf("Ignoring volumes defined in image %v because IgnoreImageDefinedVolumes is set", image.ID)
	}

	// 初始化容器相关信息，如 volumn 挂载项可能需要包含将宿主主机的一些代配置目录，
  // 如:/etc/hostname、/etc/hosts、/etc/resolv.conf 或 /dev/shm
	spec, err := c.buildContainerSpec(
		platform,
		id,
		sandboxID,
		sandboxPid,
		sandbox.NetNSPath,
		containerName,
		containerdImage.Name(),
		config,
		sandboxConfig,
		&image.ImageSpec.Config,
		volumeMounts,
		ociRuntime,
	)

	// 容器IO设置
	containerIO, err := cio.NewContainerIO(id,
		cio.WithNewFIFOs(volatileContainerRootDir, config.GetTty(), config.GetStdin()))


	// 创建一个容器。重点
	var cntr containerd.Container
	if cntr, err = c.client.NewContainer(ctx, id, opts...); err != nil {
		return nil, fmt.Errorf("failed to create containerd container: %w", err)
	}


	// 容器信息存储
	container, err := containerstore.NewContainer(meta,
		containerstore.WithStatus(status, containerRootDir),
		containerstore.WithContainer(cntr),
		containerstore.WithContainerIO(containerIO),
	)
	// Add container into container store.
	if err := c.containerStore.Add(container); err != nil {
		return nil, fmt.Errorf("failed to add container %q into store: %w", id, err)
	}

	// 后续操作
	err = c.nri.PostCreateContainer(ctx, &sandbox, &container)
```

对容器的创建操作是通过调用  [p.client.NewContainer()](https://github.com/containerd/containerd/blob/0cf48bab2cee394ccefcc05e7025ade76d142bc4/client/client.go#L279-L302) 来实现的, 这里 `p.client` 是 `containerd` gRPC客户端，而其内部又调用了 

```go
func (c *Client) NewContainer(ctx context.Context, id string, opts ...NewContainerOpts) (Container, error) {
	ctx, done, err := c.WithLease(ctx)
	if err != nil {
		return nil, err
	}
	defer done(ctx)
  
	container := containers.Container{
		ID: id,
		Runtime: containers.RuntimeInfo{
			Name: c.runtime,
		},
	}
	for _, o := range opts {
		if err := o(ctx, c, &container); err != nil {
			return nil, err
		}
	}
  
  // gRPC Call
	r, err := c.ContainerService().Create(ctx, container)
	if err != nil {
		return nil, err
	}
	return containerFromRecord(c, r), nil
}
```

这里 `c.ContainerService()` 实现了 `containers.Store`  接口，定义

```go
// https://github.com/containerd/containerd/blob/b61988670c7c951a0bf4f11f6ea9d926f3e79d35/containers/containers.go

// Store interacts with the underlying container storage
type Store interface {
	// Get a container using the id.
	//
	// Container object is returned on success. If the id is not known to the
	// store, an error will be returned.
	Get(ctx context.Context, id string) (Container, error)

	// List returns containers that match one or more of the provided filters.
	List(ctx context.Context, filters ...string) ([]Container, error)

	// Create a container in the store from the provided container.
	Create(ctx context.Context, container Container) (Container, error)
  
  // Update the container with the provided container object. ID must be set.
	//
	// If one or more fieldpaths are provided, only the field corresponding to
	// the fieldpaths will be mutated.
	Update(ctx context.Context, container Container, fieldpaths ...string) (Container, error)

	// Delete a container using the id.
	//
	// nil will be returned on success. If the container is not known to the
	// store, ErrNotFound will be returned.
	Delete(ctx context.Context, id string) error
}
```

对于gGRPC 方法对应的proto定义为

```protobuf
// https://github.com/containerd/containerd/blob/5fdf55e493d68079d22a08de1a7afe522e5838e5/api/services/containers/v1/containers.proto

service Containers {
	rpc Get(GetContainerRequest) returns (GetContainerResponse);
	rpc List(ListContainersRequest) returns (ListContainersResponse);
	rpc ListStream(ListContainersRequest) returns (stream ListContainerMessage);
	rpc Create(CreateContainerRequest) returns (CreateContainerResponse);
	rpc Update(UpdateContainerRequest) returns (UpdateContainerResponse);
	rpc Delete(DeleteContainerRequest) returns (google.protobuf.Empty);
}
```

对 `containers.Store` 接口的实现为

```go
// https://github.com/containerd/containerd/blob/261e01c2acc1eb03ac15bda80c9230a0d0dcc722/client/containerstore.go

type remoteContainers struct {
	client containersapi.ContainersClient
}

var _ containers.Store = &remoteContainers{}

// NewRemoteContainerStore returns the container Store connected with the provided client
func NewRemoteContainerStore(client containersapi.ContainersClient) containers.Store {
	return &remoteContainers{
		client: client,
	}
}

func (r *remoteContainers) Create(ctx context.Context, container containers.Container) (containers.Container, error) {
	created, err := r.client.Create(ctx, &containersapi.CreateContainerRequest{
		Container: containerToProto(&container),
	})
	if err != nil {
		return containers.Container{}, errdefs.FromGRPC(err)
	}

	return containerFromProto(created.Container), nil

}

```

上面是创建容器流程时 gRPC Client 的实现。



下面我们看下 gRPC Server 的实现，其位于为 `services/containers/service.go`, 其底层是由文件 [local.go](https://github.com/containerd/containerd/blob/9db21401c4b54f695001705423aad686407618f5/services/containers/local.go) 来实现的。

对于 Create() 方法服务端实现源码

```go
func (l *local) Create(ctx context.Context, req *api.CreateContainerRequest, _ ...grpc.CallOption) (*api.CreateContainerResponse, error) {
	var resp api.CreateContainerResponse

	if err := l.withStoreUpdate(ctx, func(ctx context.Context) error {
		container := containerFromProto(req.Container)

    // l.Store.Create() 创建
		created, err := l.Store.Create(ctx, container)
		if err != nil {
			return err
		}

		resp.Container = containerToProto(&created)

		return nil
	}); err != nil {
		return &resp, errdefs.ToGRPC(err)
	}
  
	...

	return &resp, nil
}
```

这里 `l.Store` 和客户端侧一样，它们实现的是同个接口。



 

参考资料

- https://github.com/containerd/containerd/tree/main/runtime/v2#architecture

- https://www.jianshu.com/p/d8f6c40280f8
