## Kubernetes网络模型

集群中每个Pod都会获得一个独一无二的跨整个集群的IP地址，这意味着你无需显式地在Pod之间创建连接， 而且几乎不需要处理将容器端口映射到主机端口的问题。

这创造了一个清晰，向后兼容的模型，从端口分配，命名，服务发现，负载均衡，应用配置到迁移等方面，可以将Pod与虚拟机或物理主机类比对待。

Kubernetes对于任何网络实现都有以下基本要求（除非有意为之的网络分割策略）:

- Pod可以在任何节点上与所有其他节点上的Pod进行通信，无需经过网络地址转换（NAT）。
- 节点上的代理（比如守护程序，kubelet）可以该节点上的所有Pod进行通信。

> 注意：对于那些支持在主机网络中运行Pod的平台（比如Linux），当Pod连接到节点的主机网络时，它们仍然可以在所有节点上与所有Pod进行通信，无需进行网络地址转换（NAT）。



这个模型不仅整体上更简单，而且与Kubernetes旨在实现从虚拟机（VM）迁移到容器的低摩擦迁移的愿望基本兼容。如果你的工作之前在虚拟机中运行，那么你的虚拟机有一个IP，可以与项目中的其他虚拟机通信。这与基本的模型相同。

Kubernetes的IP地址存在于Pod的范围内 - Pod中的容器共享它们的网络命名空间 - 包括它们的IP地址和Mac地址。这意味着Pod内的容器可以通过localhost相互访问各自的端口，这也意味着Pod内的容器必须协调端口的使用，但这与VM中的进程没有什么不同，这被称为“每个Pod一个IP”模型。



Kubernetes网络解决了以下四个问题：

- Pod内的容器通过回环（loopback）进行通信。
- 集群网络提供了不同Pod之间通信的能力。
- Service允许将在Pod中运行的应用程序公开，使其可以从集群外部访问。
  - Ingress provides extra functionality specifically for exposing HTTP applications, websites and APIs.
- You can also use Services to publish services only for consumption inside your cluster.



后续主要讲解以下几个方面

- Service
- Ingress
- Ingress Controllers
- EndpointSlices
- Network Policies
- DNS for Services and Pods
- IPv4/IPv6 dual-stack
- Service CluterIP allocation
- Service Internal Traffic Policy



