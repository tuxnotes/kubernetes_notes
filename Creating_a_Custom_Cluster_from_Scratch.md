# Creating a Custom Cluster from Scratch

本指导适用于想自定义Kubernetes集群的用户，如果你能在https://v1-12.docs.kubernetes.io/docs/setup/中找到一个满足你需求的方案，建议采用，因为你能从其他人的经验里获得好处。但是如果 你对IaaS, networking, configuration management, or operating system有特定要求，上面连接中已经存在的安装指导可能就不使用于你了。本指南为你提供一个安装步骤的大纲。需要指出的是自定义安装需要很大的功夫。

同时本指南对于想以更深的角度理解已运行集群安装过程中生成的脚本也是非常有帮助的。

## 1 Designing and Preparing

### 1.1 Learning

1. 你应该对kubernetes的使用应该熟悉了。建议你通过其他的Getting Started Guides搭建一个临时的集群，这有助于你熟悉CLI(kubectl)和concepts(pods, services, etc).
2. 你应该在你的客户端安装了`kubectl`工具

### 1.2 Cloud Provider

Kubernetes有一个Cloud provider的概念，Cloud Provider是一个模块，提供了一个接口，用于管理TCP Load Balancers，Nodes(instances) and Networking Routes. 这个接口定义在`pkg/cloudprovider/cloud.go`中。在没有实现cloud provider功能的环境下(for example if using bare-metal)创建自定义集群也是可以的，and not all parts of interface need to be implemented, depending on how flags are set on various components.

### 1.3 Nodes

- 可以使用虚拟机或物理主机
- 你可以搭建只有一台机器的集群，为了运行和测试所有的示例，你至少需要4个节点
- 许多安装指南在master nodes and regular nodes数量的配置不同，这并不是严格要求的
- 节点需要运行Linux x86_64架构的发行版，本指南对采用其他的操作系统和架构并没有帮助作用。
- Apiserver and etcd together are fine on a machine with 1 core and 1GB RAM for cluster with 10s of nodes. Larger or more active clusters may benefit from more cores
- 其他节点可以有任意合理数量的内存和CPU核心。这些节点也没必要在内存和核心上有相同的配置

### 1.4 Network

#### 1.4.1 Network Connectivity网络连通性

kubernetes的网络模型很有特色。

kubernetes为每个pod分配一个IP地址。在搭建集群的过程中你需要分配IP地址段，供kubernetes创建Pod的时候供Pod使用的IP地址。最简单的方式是：节点加入集群时，为节点分配不同的IP地址段。一个Pod中的进程能通过另一个Pod的IP与其通信，这种网络连通性可以采用如下两种方案实现：

1. Using an overlay network
   - An overlay network obscures the underlying network architecture from the pod network through traffic encapsulation(for example vxlan) . overlay network掩盖了底层网络架构，将Pod网络数据进行封装
   - Encapsulation reduces performance, though exactly how much depends on your solution封装会降低性能，具体性能能降低多少取决于你用的封装方案。
2. Without an overlay network
   - Configure the underlying network fabric(switches,routers,etc.) to be aware of pod IP addresses设置底层网络设备(交换机，路由器等)能够发现Pod的IP地址
   - This does not require the encapsulation provided by an overlay, and so can achieve better performance这种方案不需要overlay提供网络封装，所以能获得较好的性能

采用哪种方案，取决于你的环境和要求。目前已有一些其他方法用于实现上面的方案：

