---
title: K8S网络
date: 2024-11-20 16:46:06
tags: K8S
---

### 网络模型

Kubernetes 的网络模型假定了所有 Pod 都在一个可以直接连通的扁平的网络空间中，这在GCE(Google Compute Engine)里面是现成的网络模型，Kubernetes 假定这个网络已经存在。而在私有云里搭建 Kubernetes 集群，就不能假定这个网络已经存在了。我们需要自己实现这个网络假设，将不同节点上的 Docker 容器之间的互相访问先打通，然后运行 Kubernetes.

### 网络模型原则

- 任意节点上的Pod可以在不借助NAT的情况下与任意节点上的任意Pod进行通信；
- 节点上的代理（诸如系统守护进程、kubelet等）可以在不借助NAT的情况下与该节点上的任意Pod进行通信；
- 处于一个节点的主机网络中的Pod可以在不借助NAT的情况下与任意节点上的任意Pod进行通信（当且仅当支持Pod运行在主机网络的平台上，比如Linux等）；
- 不论在Pod内部还是外部，该Pod的IP地址和端口信息都是一致的。

在上述要求得到保证后，Kubernetes网络主要聚焦于两个任务—**IP地址管理和路由**，并致力于解决如下问题：

- 同一个Pod内多个容器之间如何通信
- 同一个Node节点中多个Pod之间如何通信
- 不同Node节点上的多个Pod之间如何通信
- Pod和Service之间如何通信
- Pod和集群外的实体如何通信
- Service和集群外的实体如何通信

### CNI

Kubernetes刚开始仅关注和负责容器编排领域的相关事宜，网络管理并不是它最核心的工作。起初，Kubernetes通过开发Kubenet来实现网络管理功能以提供满足要求的集群网络。Kubenet是一个非常简单、基础的网络插件实现。但它本身并不支持任何跨节点之间的网络通信和网络策略等高级功能，且仅适用于Linux系统，所以Kubernetes试图找到一个更优秀的方案来替代Kubenet。为了解决这个问题，CoreOs公司和Docker各自推出了**CNI**（Container Network Interface）和**CNM** (Container Network Model）规范，CNI以其完善的规范和优雅的设计击败了CNM，并成为了Kubernetes首选的网络插件接口规范。

CNI的基本思想是在容器运行时环境中创建容器时，先创建好网络命名空间（netns），然后调用CNI插件为这个网络命名空间配置网络，之后再启动容器内的进程。CNI通过Json Schema定义了容器运行环境和网络插件之间的接口声明，描述当前容器网络的配置和规范，尝试通过一种普适的方式来实现容器网络的标准化 。它仅专注于在创建容器时分配网络资源（IP、网卡、网段等）和在容器被回收时如何删除网络资源两个方面的能力。`CNI`作为Kubernetes和底层网络之间的一个抽象存在，屏蔽了底层网络实现的细节、实现了Kubernetes和具体网络实现方案的解耦，同时也克服了Kubenet不能实现跨主机容器间的相互通信等不足和短板。

借助 CNI 标准，Kubernetes 可以实现容器网络问题的解决。通过插件化的方式来集成各种网络插件，实现集群内部网络相互通信，只要实现CNI标准中定义的核心接口操作ADD，将容器添加到网络;DEL从网络中删除一个容器;CHECK，检查容器的网络是否符合预期等)。CNI插件通常聚焦在容器到容器的网络通信。

![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/k8s/K8s-CNI-Structure.png)

![3e21950f-e076-42b4-8f3b-8605e79aee3b.jpg](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/k8s/3e21950f-e076-42b4-8f3b-8605e79aee3b.jpg)

CNI的接口并不是指 HTTP，gRPC 这种接口，CNI接口是指对可执行程序的调用(exec)可执行程序,Kubernetes 节点默认的 CNI 插件路径为/opt/cni/bin

![image-20241120180423085](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/k8s/image-20241120180423085.png)

CNI通过JSON格式的配置文件来描述网络配置，当需要设置容器网络时，由容器运行时负责执行CNI插件，并通过CNI插件的标准输入（stdin）来传递配置文件信息，通过标准输出（stdout）接收插件的执行结果。从网络插件功能可以分为五类：

**Main: interface-creating**

- bridge: 创建一个桥接网络，并将宿主机和容器加入到这个桥接网络中
- ipvlan: 在容器中加入一个[ipvlan](https://www.kernel.org/doc/Documentation/networking/ipvlan.txt)接口
- loopback: 设置环回接口的状态为up状态
- macvlan: 创建一个新的mac地址，并将所有到该地址的流量转发到容器
- ptp: 创建一个新的veth对
- vlan: 分配一个vlan设备
- host-device: 将宿主机现有的网络接口移到容器内。

**IPAM: IP address allocation**

- dhcp: 在宿主机上运行一个daemon进程并代表容器发起DHCP请求。
- host-local: 维护一个已分配IP的本地数据库
- static: 向容器分配一个静态的IPv4/IPv6地址，这个地址仅用于调试目的。

**Meta: other plugins**

- flannel: 根据flannel配置文件生成一个网络接口
- tuning: 调整一个已有网络接口的sysctl参数
- portmap: 一个基于iptables的端口映射插件，将宿主机地址空间的端口映射到容器中
- bandwidth: 通过流量控制工具tbf来实现带宽限制
- sbr: 为接口配置基于源IP地址的路由
- firewall: 一个借助iptables或者firewalld来添加规则来限制出入容器流量的防火墙插件。

**Windos specific**

- win-bridge: 一个桥接插件，用于在 Windows 环境中将容器连接到宿主机网络，支持通过 NAT 实现容器与外部网络的通信。

- win-overlay: 一个覆盖网络插件，支持在 Windows 环境下跨主机创建虚拟网络，允许容器通过 VXLAN 技术与其他节点上的容器进行通信。

**容器网络插件**

![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/k8s/cni-plugins-20240717.png)

### Pod启动网络流程



![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/K8s%E7%BD%91%E7%BB%9C%E5%9B%BE.png)



### Pod 调度与启动流程

1. Pod 被调度到容器集群的某个节点上
   1. Kubernetes 调度器根据调度算法选择一个合适的节点，将 Pod 分配到该节点。
   2. 将调度决策写入 `kube-apiserver`，更新 Pod 的 `NodeName` 字段。
2. 节点的 kubelet 通过调用 CRI 插件来创建 Pod
   1. kubelet 获取 Pod 信息：
   2. kubelet`通过 watch 监听`kube-apiserver`，检测到有新 Pod 调度到本节点。
   3. 拉取 Pod 定义，包括容器配置、资源需求等。
   4. 调用 CRI 插件：
   5. 使用 gRPC 接口与 CRI 插件（如 containerd）通信，发起 Pod 创建请求。
3. CRI 插件创建 Pod 的 Sandbox 和网络命名空间
   1. CRI 插件调用容器运行时创建一个 Pause 容器（作为 Pod 的 Sandbox 容器）。
   2. 创建 Pod 的独立网络命名空间（NetNS）用于网络隔离。
   3. 生成一个唯一的 Sandbox ID 标识该 Pod 沙箱。
4. CRI 插件通过网络命名空间和 Sandbox ID 调用 CNI 插件
   1. CRI 插件通过 `RunPodSandbox` API 调用 CNI 插件，将网络命名空间和 Sandbox ID 作为参数传递。
   2. 开始为 Pod 配置网络。
5. CNI 配置 Pod 网络
   1. Flannel 为 Pod 分配一个 IP 地址，并设置节点间的网络隧道（如 VXLAN）。
   2. **Bridge CNI 插件**：在宿主机上创建一个虚拟网桥（如 `cni0`），并将 Pod 的虚拟网卡（veth pair）连接到网桥。
   3. **主机 IPAM CNI 插件**：分配具体的 IP 地址，设置路由和 DNS 信息。
   4. **返回结果**：CNI 插件返回成功状态，包含分配的 Pod IP 地址。
6. 创建 Pause 容器，并将其添加到 Pod 的网络命名空间
   1. Pause 容器作为 Pod 的基础，持有 Pod 的网络命名空间和其他资源隔离设置。
   2. 所有应用容器共享 Pause 容器的网络命名空间。
   3. Pause 容器被启动，并加入网络命名空间。
7. kubelet 调用 CRI 插件拉取应用容器镜像
   1. `kubelet` 通过 CRI 插件发起请求，确保应用容器所需的镜像可用。
   2. 检查本地镜像缓存，如果不存在则拉取镜像。
8. 容器运行时 containerd 拉取应用容器镜像
   1. 容器运行时（如 containerd）从镜像仓库（如 Docker Hub、Harbor）拉取应用容器镜像。
   2. 将镜像存储到本地镜像仓库。
9. kubelet 调用 CRI 插件来响应应用容器
   1. `kubelet` 使用 CRI 插件，发起应用容器创建请求。
   2. 配置应用容器的运行参数（启动命令、环境变量、资源限制等）。
10. CRI 插件调用容器运行时 containerd 来启动和配置应用容器
    1. CRI 插件通过 containerd 启动应用容器。
    2. 配置应用容器的 Cgroups 和 Namespace：
    3. 加入 Pod 的网络命名空间（共享 Pause 容器的 NetNS）。
    4. 配置资源隔离（CPU、内存）。
11. 应用容器被运行在 Pod 的沙箱环境中，完成整个流程。

#### 总结

- `Scheduler`：分配 Pod 到节点。
- `kubelet`：协调 CRI 和 CNI，完成 Pod 和容器的创建。
- `CRI 插件`：负责沙箱和容器的创建。
- `CNI 插件`：负责网络配置。
- `容器运行时`：负责容器镜像拉取和运行。
