# 1 Pod Overview

**`Pod`** 是kubernetes对象模型中最小的可部署对象。

## 1.1 Understanding Pods

A Pod is the basic building block of Kubernetes - the smallest and simplest unit in the Kubernetes object model that you create or deploy. Pod 是kubernetes对象模型中最小最简单的构建单元。

Pod封装了应用的容器(某些情况下是多个容器)，存储资源，唯一网络IP，以及控制容器如何运行的一些选项。

Pod代表一个部署单元：是kubernetes中一个应用的单个实例。由一个或几个紧密耦合且共享资源的容器组成。

Docker是kubernetes Pod最常见的容器运行时，但是Pod也支持其他的容器运行时。

kubernetes集群中的Pod有两种主要的使用方式：

 - **Pods that run a single container** : "一个pod一个容器"的模型是kubernetes中最常见的使用场景。这种场景下，你可以认为pod作为单个容器的封装，kubernetes管理Pod而不是容器
 - **Pods that run multiple container that need to work together**. 一个Pod可能将多个紧密结合资源共享的容器封装为一个应用。这写相互关联的容器内聚成一个服务单元——如一个容器给大家从共享存储中提供文件，而另一个独立的"sidecar"容器更新这些文件。Pod将这些容器和存储资源分装为一个独立的可管理实体。更多的使用场景请参考：
   - [The Distributed System Toolkit: Patterns for Composite Containers](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns)
   - [Container Design Patterns](https://kubernetes.io/blog/2016/06/container-design-patterns)

每个Pod意味着一个给定应用启动的单个实例，如果想水平扩展你的应用(如运行多个实例)，你应该使用多个Pod，在kubernetes中这指的是`replication` , 复制的这些容器通常通过抽象概念`Controller`作为一个组进行创建和管理。

