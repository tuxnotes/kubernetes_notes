# 3 使用kubernetes客户端

## 3.1 查看资源

```bash
# kubectl get pods,services,deployments -n NAMESPACE
# kubectl get all
```

简称：

- configmaps - cm
- daemonsets - ds
- deployments - deploy
- endpoints - ep
- events - ev
- horizontalpodautocalers - hpa
- ingresses - ing
- namespaces - ns
- nodes - no
- persistentvolumecliams - pvc
- persistentvolumes - pv
- pods - po
- replicasets - rs
- replicationcontrollers - rc
- resourcequotas - quota
- serviceaccounts - sa
- services - svc

## 3.2 删除资源

不要直接删除被监控的对象，如有部署控制的Pod等。应该先关闭它们的监控进程，或使用特定的操作删除被管理的资源。例如，可以想将一个部署缩小到0个副本(参照9.1节)，然后就可以有效地删除它监控的所有Pod了。

另外需要考虑的一点是级联删除与直接删除。如，当删除一个自定义的资源定义(CRD, 13.4节)时，其所有的依赖对象也会被删除。更多关于级联删除规则的信息参照：https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/

## 3.3 使用kubectl观察资源的变换

```bash
# kubectl get pods --watch
# watch kubectl get pods
```

## 3.4 使用kubectl编辑资源

```bash
# kubectl run nginx --image=nginx
# kubectl edit deploy/nginx
```

## 3.5 通过kubectl解释资源和字段

```bash
# kubectl explain svc
# kubectl explain svc.spec.externalIP
```

# 4 创建与修改基础工作负载

## 4.1 通过kubectl run 创建部署

kubectl run 命令是即可创建部署的清单文件的生成器。

```bash
# kubectl run ghost --image=ghost:0.9
# kubectl get deploy/ghost
```

kubectl run命令有许多选项用与设置部署参数：

- --env , 设置环境变量
- --port  定义容器端口
- --command 定义运行的命令
- --expose 自动创建一个关联的服务
- --replocas 定义pod的数量

```bash
$ kubectl run ghost --image=ghost:0.9 --port=2368 --expose # 在端口2368上启动Ghost，并随之创建一个服务
$ kubectl run mysql --image=mysql:5.5 --env=MYSQL_ROOT_PASSWORD=root 
$ kubectl run myshell --image=busybox --command -- sh -c "sleep 3600"
```

`kubectl run --help`查看各个参数的详细信息

## 4.2 通过清单文件创建对象

kubectl create 命令可以使用指向某个URL或本地文件系统中的yaml文件

```bash
# cat myns.yaml
apiVersion: v1
kind: namespace
metadata:
  name: myns
# kubectl create -f myns.yaml
# kubectl create -f https://raw.githubusercontent.com/kubernetes/kubernetes/ \
master/examples/guestbook/frontend-deployment.yaml
```

## 4.3 从零创建pod的清单文件

除非有特殊原因，否则不要单独创建Pod。应该使用一个部署对象(4.4节)监管Pod。它会通过另外一个叫做ReplicaSet(副本集)的对象监控Pod。

## 4.4 Lauching a Deployment Using a Manifest

Writing a manifest using the **Deployment** object 

```bash
# vim fancyapp.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
   name: fancyapp
 spec:
   replicas: 5
   template:
     metadata:
       labels:
         app: fancy
         env: development
     spec:
       containers:
       - name: sise
         image: mhausenblas/simpleservice:0.5.0
         ports:
         - containerPort: 9876
         env:
         - name: SIMPLE_SERVICE_VERSION
           value: "0.9"
```

things should be done when lauching the app:

- set the number of pods (replicas), or identical copies, that should be launched and supervised.
- Label it , such as with env=development
- set environment variabls such as SIMPLE_SERVICE_VERSION

**如果想要删除一个部署及其replica set 和其监管的pods, 执行kubectl delete deploy/fancyapp.单独删除Pod，Deployment会重建Pod**

Deployments allow you to scale the app(Recipe 9.1) as well as roll out a new version or roll back to a previous version. In General, **Deployments are good for stateless apps that require with identical characteristics**

deployment负责监管Pod和replicaset，可以允许你细致地控制如何以及何时应该将Pod设计到新版本或回滚到之前的版本。一般来说无需在意Deployment监管的Pod与replicaset，除非特写特殊情况，比如需要调试某个Pod。

使用replicaset代替原来的replication controller.虽然目前两者唯一的区别在于replicaset支持与集合为基础的标签与查询，但以后会有更多功能加入到replicaset，而RC将逐步退出舞台。

## 4.5 更新部署

```bash
$ kubectl run sise --image=mhausenblas/simpleservice:0.4.0
$ kubectl set image deployment sise mhausenblas/simpleservice:0.5.0
$ kubectl rollout status deployment sise
$ kubectl rollout history deployment sise
$ kubectl edit deploy sise
```

如何设置并export了环境变量KUBE_EDITOR的话，将会在指定的编辑器中打开当前部署。

```bash
$ kubectl rollout history deployment sise
deployments "sise"
REVISION        CHANGE-CAUSE
1               <none>
2               <none>
3               <none>
```

kubectl rollout history deployment sise可查看历史版本。**CHANGE-CAUSE一栏为空是因为没有使用kubeclt create 的--record选项** , 如果想连接什么变更触发了新的版本，可以加上此选项。

还有很多的kubectl命令可以用于更新部署：

- kubectl apply, 根据清单文件更新部署。
- kubectl replace, 根据清单文件替换部署，与apply不同的是replace对象必须存在。
- kubectl patch，更新特定的键值

```bash
kubectl patch deployment sise -p '{"spec": {"template":
{"spec": {"containers":
[{"name": "sise", "image": "mhausenblas/simpleservice:0.5.0"}]}}}}'
```

**回滚**

```bash
$ kubectl rollout undo deployment sise --to-version=2
```

回退到版本2

可以通过`kubectl get deploy/sise -o yaml`确认端口定义已经被删除

> NOTE: 只有Pod模板(即.spec.template下的键值)的变更才可以触发新的部署。例如：环境变量，端口或容器镜像。部署方面的变更，比如副本个数等不会触发新的部署。

# 5 使用服务

Kubernetes service的具体内容如下图所示

![kubernetes service概念](./k8s_service.png)

service为一组Pod提供稳定的虚拟IP地址(Virtual IP, VIP). 即使加入新的pod 或移除已有Pod，service也能保证客户端可以通过VIP可靠地发现并连接到pod中运行的容器。VIP中的virtual的意思是说该IP并不是连接到网络接口的真实IP地址，它的目的只是为了将访问发送到一个或多个Pod。kube-proxy负责维护VIP与Pod之间的映射，集群中的每个节点都需要运行该进程。 kube-proxy进程查询API服务器以获知集群中的新服务，并相应第更新节点的iptables规则，以提供必要的路由信息。

## 5.1 Creating a Service to Expose Your Application

如何在集群内提供一个稳定可靠的方法，发现并访问应用？

通过为应用的Pod创建service来实现

```bash
$ kubectl run nginx --image=nginx
$ kubectl expose deploy/nginx --port 80
```

kubectl expose命令会创建一个service

如果想用浏览器访问该服务，那么可以在另一个中断运行代理，如下所示：

```bash
$ kubectl proxy --address='0.0.0.0'  --accept-hosts='^*$'
```

浏览器访问：http://localhost:8001/api/v1/namespaces/default/services/nginx/proxy/

同样也可以通过yaml文件创建service

```bash
$ cat svc-nginx.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    run: nginx
  ports:
  - port: 80
```

需要注意的是yaml文件中的selector ，which is used to select all the pods that make up this microservice abstraction.
