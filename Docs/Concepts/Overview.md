# 1 什么是kubernetes

Kubernetes is portable, extensible open-source platform for **managing containerized workloads and services**, that facilitates both declarative configuration and automation. 

kubernetes 是一个可移植，可扩展用于管理容器化应用和服务的开源平台。它有丰富且增长迅速的生态圈。Kubernetes由google于2014年开源。

## 1.1 为什么需要Kubernetes,它能做什么

kubernetes 具有一下几个特性：

- a container platform容器平台
- a microservices platform微服务平台
- a portable cloud platform and a lot more可移植的云平台

Kubernetes provides a **container-centric** management environment. It orchestrates computing, networking, and storage infrastructure on behalf of user workloads. This provides much of the simplicity of Platform as a Service(PaaS) with flexibility of infrastructure as a Service(IaaS), and enables protability across infrastructure providers.

kubernetes提供了一个以容器为中心的管理环境，它代表用户负载来编排计算，网络，存储等基础设施。

## 1.2 How Kubernetes is a platform

尽管kubernetes提供了很多的功能，但总有一些新的使用场景得益于kubernetes的新特性。如具体应用的开发工作流程化来提高开发速度；临时编排功能起始阶段需要的高度自动扩容能力。kubernetes被设计为服务于平台，建立强大的生态时期部署，扩容，管理等更加容易。

`Labels`赋予用户组织资源更强大的能力。`Annotations`使用户更加容易的通过自定义信息对资源进行丰富，限制，或修饰。并且`Kubernetes control plane`为开发者和用户提供了相同的API，用户可以使用自定义API编写自己的controller, schedulers等

## 1.3 What Kubernetes is not

Kubernetes不是传统意义上包含一切的PaaS系统。因为kubernetes是操作与容器层面，而不是硬件层面。kubernetes为PaaS提供了常用特性，如果部署，扩容，负载均衡，日志，监控。但kubernetes不是一个整块，默认这些功能都是可选的，可插拔的。kubernetes为创建开发平台提供了基础，但是留给用户更多的选择性和灵活性来决定哪里是其关注的重点。

Kubernetes特点：

 - 不限定其支持的应用的类型。kubernetes的目标是支持多种多样的工作负载，包括无状态的，有状态的，数据处理等类型的工作负载。如果应用能在容器中运行，则在kubernetes上运行也会很好。
 - kubernetes不部署源代码，不编译你的应用。持续集成，持续部署根据组织文化，偏好，技术需求等不同而不同。
 - 不提供应用层面的服务，如中间件(e.g. message buses),数据处理框架(如Spark),数据库(如mysql),缓存，或集群存储系统(如Ceph)等作为内置服务。这些组件可以运行于kubernetes，且运行于kubernetes上的应用可以通过portable mechanisms，如Open Service Broker进行访问
 - 不致力于日志，监控，告警等解决方案。但提供了一些集成来说明这些方案数据收集和获取的概念和机制。
 - 不提供也不强制使用的配置语言或系统(如jsonnet)。提供了声明式 的API
 - 不提供也不采用任何综合性的机器配置，维护，管理或自愈系统。

kubernetes不仅仅是一个编排系统。事实上，它消除了编排的需要。编排的技术定义是执行一个工作流程：先做A，再做B，接着做C。相反，kubernetes由一系列相互独立，可组合的控制过程组成，它能持续的将当前状态转变到期望的状态。不需要关心是如果从A到C的。集中化的控制也不是必须的。这就使系统更加容易的使用，且更强大，弹性和扩展。

## 1.4 Why containers

容器的好处：

 - **应用的创建和部署更加敏捷**: 对比于VM镜像，容器镜像的创建更加容易和高效
 - **持续部署持续继承持续交付**：提供了更加可靠和更高频率的镜像构建和部署，且由于镜像的immutability，回滚操作更加容易和快速
 - **Dev and Ops separation of concerns**: Create application container images at build/release time rather than deployment time, thereby decoupling applications from infrastructure.
 - **Observability** Not only surfaces OS-level information and metrics , but also application health and other signals.不仅监控系统层面的信息和数据，还包括应用的健康状态和其他信息
 - **保证了开发，测试，生产环境的一致性**
 - Cloud and OS distribution portability
 - 以应用为中心的管理
 - 低耦合，分布式，弹性，微服务
 - 提高资源隔离
 - 提高资源利用率

# 2 Kubernetes组件

部署一个kubernetes集群需要以下组件：

 - Master Components
 - Node Components
 - Addons

## 2.1 Master Components

Master组件提供了集群的控制面板。Master组件为集群提供全局的控制(如调度)，检测并响应集群事件(starting up a new pod when a replication controller's **replicas** field is unsatisfied)。Master组件可以运行于集群中的任何机器上，但出于简单考虑，部署脚本一般将Master的所有组件部署于同一台机器上，并不在这台机器上运行容器。

See [Building High-Availability Clusters](https://kubernetes.io/docs/admin/high-availability/) for an example multi-master-VM setup.

### 2.1.1 kube-apiserver

kube-apiserver是Master上的组件。kube-apiserver暴露Kubernetes API. 是Kubernetes control plane的前端。其被设计为可水平扩展，所以能部署为多个实例。

### 2.1.2 etcd

作为kubernetes集群数据的后端存储，保证了key value存储数据的一致性和高可用。kubernetes集群的etcd数据必须有备份计划。

### 2.1.3 kube-scheduler

kube-scheduler是Master组件。kube-scheduler监视哪些新创建的pod还没有节点来运行它们，并选择几点来运行pod

调度策略需要考虑介个因素，包括单个的和整体的资源要求，硬件/软件/策略约束，亲和性和隔离性(同一个节点不能部署两个相同的)规定，数据本地化，inter-workload interference and deadlines.

### 2.1.3 kube-controller-manager

Master组件会运行多个controller进程。逻辑上将，每个controller是一个独立的进程，但为了减小复杂性，这些controller进程被编译到一个二进制文件并在单个进程中运行。

这些controller包括以下集中controller:

 - Node Controller: Responsible for noticing and responding when nodes go down.
 - Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
 - Endpoints Controller: Populates the Endpoints object(that is, joins Services & Pods)
 - Service Account & Token Controllers: Create default accounts and API access for tokens for new namespaces.

### 2.1.4 cloud-controller-manager

cloud-controller-manager runs controllers that interact with the underlying cloud providers.

## 2.2 Node Components

Node components运行于每个node,维护运行中的pods，并为kubernetes提供运行时环境。

### 2.2.1 kubelet

kubelet是运行于每个node上的agent，It makes sure that containers are running in a pod.

The kubelet takes a set of PodSpecs that are provided through various mechanisms and **ensure that the containers described in those PodSpecs are running and healthy**. **The kubelet doesn't manage containers which were not created by Kubernetes**.

### 2.2.2 kube-proxy

kube-proxy enables the Kubernetes **service abstraction by maintaining network rules on the host and performing connection forwarding**.

### 2.2.3 Container Runtime

容器运行时是一个软件，负责运行容器。kubernetes支持几种容器运行时软件：Docker, containerd, cri-o, rktlet and any implementation of kubernetes CRI(Container Runtime Interface)

## 2.3 Addons附件

