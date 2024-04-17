---
title: kubernetes中的overlay网络与underlay网络的区别
date: 2024-04-10T16:04:36+08:00
type: post
toc: true
url: /posts/overlay-vs-underlay-in-kubernetes-network
categories:
  - 程序开发
tags:
  - k8s
  - overlay
  - underlay
---

Kubernetes 中的 overlay 网络和 underlay 网络是两个不同的网络层面。

# Underlay 网络

在 Kubernetes 网络架构中，Underlay 网络是指承载 Kubernetes 网络流量的物理网络或底层网络。这个网络通常由物理交换机、路由器和其他网络硬件组成，它们之间通过各种路由协议（例如 OSPF、BGP 等）连接在一起组成的传统网络。

Underlay 网络负责为 Kubernetes 节点提供基本的网络连接，它为上层的 overlay 网络提供支持。

总之，Kubernetes 的网络流量，例如 Pod 到 Pod、Pod 到 Service 等都将在这个 Underlay 网络上进行传输。

# Overlay 网络

对于 `Overlay` 网络也被称为 `覆盖网络`，想必只要接触过一点 kubernetes 网络知识的同学都不陌生，它主用来解决 **不同节点**  中 **Pod** 之间通讯的一种网络解决方案。
![Overlay](https://blogstatic.haohtml.com//uploads/2024/04/image_1chvp7s4g15n0134iv1tvag1mkd8g.png)

在上图可以看到 `Overlay` 网络是构建在 `underlay` 网络之上的一层虚拟网络，它通过封装数据包（如VXLAN）通过物理网络进行传输，到达目标网络后再对数据包进行解包得出真正的目标地址，最后响应数据包原路返回，从而实现跨网段的网络通讯。

它就像创建了一个逻辑网络，可以实现跨越不同节点和集群实现网络正常通讯，但有一个前提是对于 `overlay` 网络要求节点网络三层互通。对于  overlay 不好的一点是存在封装、解封装过程，带来一定性能损耗，也因此各大云厂商均大力发展 `underlay`容器网络，使得容器网络具有直通能力，比如ELB直通容器，容器直接绑eip等。

常见的 `overlay` 网络有 `Flannel`、`Calico`、`Weave` 等。

总之，`Overlay` 网络为 Kubernetes 集群中的 Pod 提供 IP 地址,并实现 Pod 之间的通信。

# 这两者的关系

- `Underlay` 网络提供底层的物理网络连接。
- `Overlay` 网络则基于 underlay 网络构建一个逻辑网络，用于保证k8s网络中不同节点上面的 Pod 之间的通信。
- `Overlay` 网络利用封装技术在底层 `underlay` 网络上传输数据包。

总的来说，underlay 网络负责节点之间的基本网络连接，而 overlay 网络则负责为 Kubernetes 中的 Pod 提供网络功能和互联互通。两者相互配合,为 Kubernetes 集群提供完整的网络支持。

# Non-overlay 

`Non-overlay` 网络指的是不使用 overlay覆盖网络技术,而是直接利用底层主机的网络来实现 Kubernetes 集群中 Pod 的通信。

在 `non-overlay` 模式下，Kubernetes 集群中的每个 Pod 都直接连接到底层主机网络，没有额外的虚拟网络层。Pod 可以直接使用底层网络提供的 IP 地址进行通信，不需要进行额外的封包和解包过程。

常见的 non-overlay 网络解决方案包括:

1. **Host-Gateway**:
   - 利用 Linux 主机上的路由表将 Pod 的流量直接发送到目的地。
   - 需要底层网络支持直接路由传输。

2. **Macvlan**:
   - 为每个 Pod 分配一个虚拟网卡，直接连接到底层主机网络。
   - Pod 就像直接连在底层物理网络中一样。

3. **SRIOV(Single Root I/O Virtualization)**:
   - 通过硬件虚拟化技术,让 Pod 直接使用物理网卡，绕过虚拟化开销。
   - 需要支持 SR-IOV 的网卡。

## Non-overlay 与 overlay 的区别

相较于 overlay 网络，non-overlay 网络通常具有更好的性能,因为数据包无需经过额外的封包和解包过程。但它也需要底层网络满足特定要求,且灵活性和隔离性较差。

总的来说，overlay 和 non-overlay 网络各有优缺点，在部署时需要根据具体的网络要求和环境进行选择。

## Non-overlay与Underlay 的区别

Non-overlay 与 underlay 网络虽然都指底层物理网络基础设施，但它们在 Kubernetes 中的含义和用途有一些区别：

**Underlay 网络**

- Underlay 指的是整个 Kubernetes 集群的底层物理网络基础设施，比如交换机、路由器等设备。
- Underlay 网络为 Kubernetes 节点提供基本的网络连接，是 overlay 网络的基础。
- Overlay 网络仍然需要在 underlay 网络之上运行，通过隧道和封包技术传输数据。

**Non-overlay 网络**

- Non-overlay 是指不使用额外的 overlay 网络,而是直接将 Pod 连接到底层主机网络。
- Pod 的通信直接在主机网络上进行,不需要额外的封包和解包。
- Non-overlay 模式下,Pod 可以直接使用底层网络提供的 IP 地址进行通信。

**主要区别**

1. **层级关系**:
    - Underlay 是最底层的物理网络基础设施。
    - Overlay 网络工作在 underlay 之上的虚拟网络层。
    - Non-overlay 则直接在 underlay 网络上运行，不需要额外的虚拟化层。

2. **工作原理**:
    - Overlay 网络需要封装和解包数据包,通过隧道在 underlay 网络上传输。
    - Non-overlay 直接在 underlay 网络上传输数据,无需额外封包和解包。

3. **灵活性**:
    - Overlay 网络通常更加灵活,可跨节点和集群。
    - Non-overlay 受限于底层网络拓扑,灵活性较差。

总的来说，`underlay` 指的是物理网络基础设施，是所有解决方案的基础；而 `overlay` 和 `Non-overlay` 则是在这个 underlay 基础之上为 Pod 提供网络服务的两种不同方式，前者通过虚拟网络，后者直接利用物理网络。

# 参考资料

- https://zhuanlan.zhihu.com/p/682589887
- https://blog.csdn.net/xu710263124/article/details/134312420
