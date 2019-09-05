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

1. Use a network plugin which is called by kubernetes
   - Kubernetes support the CNI network plugin interface Kubernetes支持CNI网络插件接口
   - 这里有一些为kubernetes提供插件服务的解决方案(按字母)：Calico, Flannel, Open vSwitch(OVS), Romana, Weave, 自己编写
2. Compile support directly into Kubernetes
   - This can be done by implementing the "Routes" interface of a Cloud Provider module
   - The GCE and AWS guides use this approach
3. Configure the network external to Kubernetes设置kubernetes的外部网络
   - This can be done by manually running commands, or through a set of externally maintained scripts
   - You have to implement this yourself, but it can give you an extra degree of flexibility

你需要给Pod选择一个IP地址段

有多种方法：

- GCE：each project 有自己的地址段`10.0.0.0/8`. 从此地址段截取`/16`地址段给kubernetes集群使用。每个节点会得到一个子地址段
- AWS: 整个organization使用一个VPC, 每个集群从VPC中截取一段IP地址，或者是不同的集群使用不同的VPC



给每个节点的Pod分配一个CIDR子网，或者是一个较大范围的CIRD，其小范围的CIDR自动分配给每个节点

- 你需要计算总的IP数：每个节点的最大Pod数量 * 节点的最大数量。通常的选择是每个节点分配一个`/24`的网络，支持254个Pod。如果IP地址稀少，可以采用`/26` (每个机器62个Pod)或`/27`(每个节点30个Pod)
- 例如，使用`10.10.0.0/16`作为集群的地址范围，每个节点分别使用的IP地址为从`10.10.0.0/24`到`10.10.255.0/24`,多达256个。
- 需要使用overlay使这些地址能路由或连通

kubernetes还需要给每个service分配一个IP地址。但service的IP地址不要求必须可路由到。`kube-proxy`负责数据包离开节点前将service ip地址转换为pod的IP地址。你的确需要为service分配一个地址段，称为`SERVICE_CLUSTER_IP_RNAGE`. 例如，你可以设置`SERVICE_CLUSTER_IP_RANGE="10.0.0.0/16"` , 运行同时激活65534个不同的service。需要注意的是：你可以扩展这个地址范围，但不能移除正在被service和pod使用的地址。

**同时需要为master节点准备一个静态IP**

- 这个IP称为`**MASTER_IP**`
- 防火墙要允许访问apiserver的80或443端口

#### 1.4.2 Network Policy

kubernetes 启用了Pod间更加精细的网络策略定义，通过使用NetworkPolicy resource实现。然而并不是所有的网络providers支持kubernetes NetworkPolicy API.

### 1.5 Cluster Naming

你需要为集群选择一个简短的名称，且随着将来集群名称的增多，要保证此名称的唯一性。集群名称在下面几种情况下会用到：

- `kubectl`用此名称来区分你访问的不同集群。将来你也可能需要第二集群用于测试新的kubernetes发布版本。
- kubernetes集群能创建cloud provider resources(例如，AWS ELBs). and different cluster need to distinguish wich resources each created. Call this `**CLUSTER_NAME**`。

### 1.6 Binary software

你需要下面二进制程序：

- etcd
- 容器运行时：docker or rkt
- kubernetes: kubelet, kubeproxy, kube-apiserver, kube-controller-manager, kube-scheduler

#### 1.6.1 Downloading and Extracting Kubernetes Binaries

推荐使用kubernetes发布的二进制程序。github上下载的tar包不在包含kubernetes的二进制文件需要执行脚本`./kubernetes/cluster/get-kube-binaries.sh`脚本下载，`./kubernetes/server/bin`包含需要的二进制文件

#### 1.6.2 Selecting Images

docker, kubelet, kube-proxy以非容器的方式运行，其他的也可以用二进制启动系统守护进程。对于etcd,kube-apiserver,kube-controller-manager,kube-scheduler推荐使用容器的方式运行。

### 1.7 Security Models

There are two main options for security:

- 通过HTTP方式访问apiserver: 需要防火墙确保安全；部署叫容易
- 通过HTTPS访问apiserver: 使用证书为用户加密，推荐这种方式，证书的配置需要技巧和智慧

下面是采用HTTPS的方式，你需要准备certs and credentials

#### 1.7.1 Preparing Certs准备证书

需要准备下面几个证书：

- The master needs a cert to act as an HTTPS server.master节点作为HTTPS服务器需要证书
- The kubelets optionally need certs to identify themselves as clieants of the master, and when serving its own API over HTTPS

Unless you plan to have a real CA generate your certs, you will need to generate a root cert and use that to sign the master, kubelet, and the kubectl certs.How to do this is described in the [authentication documentation][]







