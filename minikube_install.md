# minikube installatioin

参考: https://yq.aliyun.com/articles/221687 https://www.jianshu.com/p/18441c7434a6

考虑到网速问题，docker-ce 和kubectl kukelet kubeadm采用阿里镜像源的帮助文档下载

## 1 安装步骤

### 1.1 移除已经安装的docker

[root@dev01 ~]#  yum remove docker docker-common docker-selinux docker-engine

Loaded plugins: fastestmirror
No Match for argument: docker
No Match for argument: docker-common
No Match for argument: docker-selinux
No Match for argument: docker-engine
No Packages marked for removal
[root@dev01 ~]# uname -r
3.10.0-862.el7.x86_64
[root@dev01 ~]# cat /etc/redhat-release 
CentOS Linux release 7.5.1804 (Core) 

### 1.2 安装依赖组件

[root@dev01 ~]# yum install -y yum-utils device-mapper-persistent-data lvm2

### 1.3 下载yum仓库源配置文件

[root@dev01 ~]# wget -O /etc/yum.repos.d/docker-ce.repo https://download.docker.com/linux/centos/docker-ce.repo
-bash: wget: command not found
[root@dev01 ~]# yum -y install wget
[root@dev01 ~]# wget -O /etc/yum.repos.d/docker-ce.repo https://download.docker.com/linux/centos/docker-ce.repo
--2019-08-10 21:26:18--  https://download.docker.com/linux/centos/docker-ce.repo
Resolving download.docker.com (download.docker.com)... 13.35.125.6, 13.35.125.11, 13.35.125.20, ...
Connecting to download.docker.com (download.docker.com)|13.35.125.6|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2424 (2.4K) [binary/octet-stream]
Saving to: ‘/etc/yum.repos.d/docker-ce.repo’

100%[======================================>] 2,424       --.-K/s   in 0s      

2019-08-10 21:26:19 (25.2 MB/s) - ‘/etc/yum.repos.d/docker-ce.repo’ saved [2424/2424]

### 1.4 修改repo文件

[root@dev01 ~]# sed -i 's+download.docker.com+mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo

### 1.5 安装docker-ce

[root@dev01 ~]# yum makecache fast
[root@dev01 ~]# yum install docker-ce

### 1.6 配置下载kubernetes组件的yum源

由于从github下载比较慢，所以这里采用aliyun的kubernetes的yum源

```bash
[root@dev01 ~]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 1.7 安装kubectl

[root@dev01 ~]# yum -y install  kubectl
[root@dev01 ~]# which kubectl
/usr/bin/kubectl

### 1.8 安装minikube

[root@dev01 ~]# curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.2.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 39.8M  100 39.8M    0     0  4397k      0  0:00:09  0:00:09 --:--:-- 4626k
[root@dev01 ~]# minikube 
anaconda-ks.cfg  .bash_profile    .mysql_history   .tcshrc
.bash_history    .bashrc          .pki/            v1.15.2.tar.gz
.bash_logout     .cshrc           .rnd             .viminfo

### 1.9 配置docker daemon的registry-mirros

[root@dev01 ~]# which docker
/usr/bin/docker
[root@dev01 ~]# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: https://docs.docker.com
[root@dev01 ~]# ls /etc/ | grep docker
[root@dev01 ~]# systemctl start docker   // etc下的docker目录只有在docker启动后才有

[root@dev01 ~]# ls /etc/ | grep docker
docker
[root@dev01 ~]# cd /etc/docker/
[root@dev01 docker]# ls
key.json
[root@dev01 docker]# pwd
/etc/docker
[root@dev01 docker]# vim daemon.json  // 配置registy-mirrors
[root@dev01 docker]# systemctl daemon-reload

[root@dev01 docker]# systemctl restart docker
[root@dev01 docker]# docker info
Client:
 Debug Mode: false

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 19.03.1
 Storage Driver: overlay2
  Backing Filesystem: xfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 894b81a4b802e4eb2a91d1ce216b8817763c29fb
 runc version: 425e105d5a03fabd737a126ad93d62a9eeede87f
 init version: fec3683
 Security Options:
  seccomp
   Profile: default
 Kernel Version: 3.10.0-862.el7.x86_64
 Operating System: CentOS Linux 7 (Core)
 OSType: linux
 Architecture: x86_64
 CPUs: 1
 Total Memory: 991.7MiB
 Name: dev01
 ID: LGNP:UU7E:4Q7W:CIQS:4X5Z:5KUS:6E3C:XCON:RGDR:KF7Z:GMTL:LJSH
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Registry Mirrors:
  https://docker.mirrors.ustc.edu.cn/
  https://registry.docker-cn.com/
 Live Restore Enabled: false

### 1.10 查看minikube的帮助信息

**[root@dev01 docker]# minikube --help**
Minikube is a CLI tool that provisions and manages single-node Kubernetes clusters optimized for development workflows.

Usage:
  minikube [command]

Available Commands:
  addons         Modify minikube's kubernetes addons
  cache          Add or delete an image from the local cache.
  completion     Outputs minikube shell completion for the given shell (bash or zsh)
  config         Modify minikube config
  dashboard      Access the kubernetes dashboard running within the minikube cluster
  delete         Deletes a local kubernetes cluster
  docker-env     Sets up docker env variables; similar to '$(docker-machine env)'
  help           Help about any command
  ip             Retrieves the IP address of the running cluster
  kubectl        Run kubectl
  logs           Gets the logs of the running instance, used for debugging minikube, not user code
  mount          Mounts the specified directory into minikube
  profile        Profile gets or sets the current minikube profile
  service        Gets the kubernetes URL(s) for the specified service in your local cluster
  ssh            Log into or run a command on a machine with SSH; similar to 'docker-machine ssh'
  ssh-key        Retrieve the ssh identity key path of the specified cluster
  start          Starts a local kubernetes cluster
  status         Gets the status of a local kubernetes cluster
  stop           Stops a running local kubernetes cluster
  tunnel         tunnel makes services of type LoadBalancer accessible on localhost
  update-check   Print current and latest version number
  update-context Verify the IP address of the running cluster in kubeconfig.
  version        Print the version of minikube

Flags:
      --alsologtostderr                  log to standard error as well as files
  -b, --bootstrapper string              The name of the cluster bootstrapper that will set up the kubernetes cluster. (default "kubeadm")
  -h, --help                             help for minikube
      --log_backtrace_at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log_dir string                   If non-empty, write log files in this directory
      --logtostderr                      log to standard error instead of files
  -p, --profile string                   The name of the minikube VM being used.  
                                         	This can be modified to allow for multiple minikube instances to be run independently (default "minikube")
      --stderrthreshold severity         logs at or above this threshold go to stderr (default 2)
  -v, --v Level                          log level for V logs
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging

Use "minikube [command] --help" for more information about a command.
**[root@dev01 docker]# minikube start --help**
Starts a local kubernetes cluster using VM. This command
assumes you have already installed one of the VM drivers: virtualbox/parallels/vmwarefusion/kvm/xhyve/hyperv.

Usage:
  minikube start [flags]

Flags:
      --apiserver-ips ipSlice             A set of apiserver IP Addresses which are used in the generated certificate for kubernetes.  This can be used if you want to make the apiserver available from outside the machine (default [])
      --apiserver-name string             The apiserver name which is used in the generated certificate for kubernetes.  This can be used if you want to make the apiserver available from outside the machine (default "minikubeCA")
      --apiserver-names stringArray       A set of apiserver names which are used in the generated certificate for kubernetes.  This can be used if you want to make the apiserver available from outside the machine
      --apiserver-port int                The apiserver listening port (default 8443)
      --cache-images                      If true, cache docker images for the current bootstrapper and load them into the machine. Always false with --vm-driver=none. (default true)
      --container-runtime string          The container runtime to be used (docker, crio, containerd) (default "docker")
      --cpus int                          Number of CPUs allocated to the minikube VM (default 2)
      --cri-socket string                 The cri socket path to be used
      --disable-driver-mounts             Disables the filesystem mounts provided by the hypervisors (vboxfs, xhyve-9p)
      --disk-size string                  Disk size allocated to the minikube VM (format: <number>[<unit>], where unit = b, k, m or g) (default "20g")
      --dns-domain string                 The cluster dns domain name used in the kubernetes cluster (default "cluster.local")
      --docker-env stringArray            Environment variables to pass to the Docker daemon. (format: key=value)
      --docker-opt stringArray            Specify arbitrary flags to pass to the Docker daemon. (format: key=value)
      --download-only                     If true, only download and cache files for later use - don't install or start anything.
      --enable-default-cni                Enable the default CNI plugin (/etc/cni/net.d/k8s.conf). Used in conjunction with "--network-plugin=cni"
      --extra-config ExtraOption          A set of key=value pairs that describe configuration that may be passed to different components.
                                          		The key should be '.' separated, and the first part before the dot is the component to apply the configuration to.
                                          		Valid components are: kubelet, kubeadm, apiserver, controller-manager, etcd, proxy, scheduler
                                          		Valid kubeadm parameters: ignore-preflight-errors, dry-run, kubeconfig, kubeconfig-dir, node-name, cri-socket, experimental-upload-certs, certificate-key, rootfs, pod-network-cidr
      --feature-gates string              A set of key=value pairs that describe feature gates for alpha/experimental features.
      --gpu                               Enable experimental NVIDIA GPU support in minikube (works only with kvm2 driver on Linux)
  -h, --help                              help for start
      --hidden                            Hide the hypervisor signature from the guest in minikube (works only with kvm2 driver on Linux)
      --host-only-cidr string             The CIDR to be used for the minikube VM (only supported with Virtualbox driver) (default "192.168.99.1/24")
      --hyperkit-vpnkit-sock string       Location of the VPNKit socket used for networking. If empty, disables Hyperkit VPNKitSock, if 'auto' uses Docker for Mac VPNKit connection, otherwise uses the specified VSock.
      --hyperkit-vsock-ports strings      List of guest VSock ports that should be exposed as sockets on the host (Only supported on with hyperkit now).
      --hyperv-virtual-switch string      The hyperv virtual switch name. Defaults to first found. (only supported with HyperV driver)
      --image-mirror-country string       Country code of the image mirror to be used. Leave empty to use the global one. For Chinese mainland users, set it to cn
      --image-repository string           Alternative image repository to pull docker images from. This can be used when you have limited access to gcr.io. Set it to "auto" to let minikube decide one for you. For Chinese mainland users, you may use local gcr.io mirrors such as registry.cn-hangzhou.aliyuncs.com/google_containers (default "registry.cn-hangzhou.aliyuncs.com/google_containers")
      --insecure-registry strings         Insecure Docker registries to pass to the Docker daemon.  The default service CIDR range will automatically be added.
      --iso-url string                    Location of the minikube iso (default "https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.2.0.iso")
      --keep-context                      This will keep the existing kubectl context and will create a minikube context.
      --kubernetes-version string         The kubernetes version that the minikube VM will use (ex: v1.2.3) (default "v1.15.0")
      --kvm-network string                The KVM network name. (only supported with KVM driver) (default "default")
      --memory int                        Amount of RAM allocated to the minikube VM in MB (default 2048)
      --mount                             This will start the mount daemon and automatically mount files into minikube
      --mount-string string               The argument to pass the minikube mount command on start (default "/root:/minikube-host")
      --network-plugin string             The name of the network plugin
      --nfs-share strings                 Local folders to share with Guest via NFS mounts (Only supported on with hyperkit now)
      --nfs-shares-root string            Where to root the NFS Shares (defaults to /nfsshares, only supported with hyperkit now) (default "/nfsshares")
      --no-vtx-check                      Disable checking for the availability of hardware virtualization before the vm is started (virtualbox)
      --registry-mirror strings           Registry mirrors to pass to the Docker daemon
      --service-cluster-ip-range string   The CIDR to be used for service cluster IPs. (default "10.96.0.0/12")
      --uuid string                       Provide VM UUID to restore MAC address (only supported with Hyperkit driver).
      --vm-driver string                  VM driver is one of: [virtualbox parallels vmwarefusion kvm xhyve hyperv hyperkit kvm2 vmware none] (default "virtualbox")
      --xhyve-disk-driver string          The disk driver to use [ahci-hd|virtio-blk] (only supported with xhyve driver) (default "ahci-hd")

Global Flags:
      --alsologtostderr                  log to standard error as well as files
  -b, --bootstrapper string              The name of the cluster bootstrapper that will set up the kubernetes cluster. (default "kubeadm")
      --log_backtrace_at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log_dir string                   If non-empty, write log files in this directory
      --logtostderr                      log to standard error instead of files
  -p, --profile string                   The name of the minikube VM being used.  
                                         	This can be modified to allow for multiple minikube instances to be run independently (default "minikube")
      --stderrthreshold severity         logs at or above this threshold go to stderr (default 2)
  -v, --v Level                          log level for V logs
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging

### 1.11 启动集群

**[root@dev01 docker]# minikube start --registry-mirror=https://registry.docker-cn.com --vm-driver none**
😄  minikube v1.2.0 on linux (amd64)
✅  using image repository registry.cn-hangzhou.aliyuncs.com/google_containers
🔥  Creating none VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
🐳  Configuring environment for Kubernetes v1.15.0 on Docker 19.03.1
💾  Downloading kubeadm v1.15.0
💾  Downloading kubelet v1.15.0

^C

**由于本次启动集群需要kubeadm和kubelet组件，但下载速度较慢，所以先取消启动集群，然后先安装kubeadm 和 kubelet组件**

**[root@dev01 docker]# yum -y install kubeadm kubelet**

**[root@dev01 docker]# minikube start --registry-mirror=https://registry.docker-cn.com --vm-driver none**
😄  minikube v1.2.0 on linux (amd64)
✅  using image repository registry.cn-hangzhou.aliyuncs.com/google_containers
💡  Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
🔄  Restarting existing none VM for "minikube" ...
⌛  Waiting for SSH access ...
🐳  Configuring environment for Kubernetes v1.15.0 on Docker 19.03.1
💾  Downloading kubelet v1.15.0
🔄  Relaunching Kubernetes v1.15.0 using kubeadm ... 
🤹  Configuring local host environment ...

⚠️  The 'none' driver provides limited isolation and may reduce system security and reliability.
⚠️  For more information, see:
👉  https://github.com/kubernetes/minikube/blob/master/docs/vmdriver-none.md

⚠️  kubectl and minikube configuration will be stored in /root
⚠️  To use kubectl or minikube commands as your own user, you may
⚠️  need to relocate them. For example, to overwrite your own settings:

    ▪ sudo mv /root/.kube /root/.minikube $HOME
    ▪ sudo chown -R $USER $HOME/.kube $HOME/.minikube

💡  This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true
⌛  Verifying: apiserver proxy etcd scheduler controller dns
🏄  Done! kubectl is now configured to use "minikube"
[root@dev01 docker]# netstat -anutpl
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:43360         0.0.0.0:*               LISTEN      3162/kubelet        
tcp        0      0 127.0.0.1:10248         0.0.0.0:*               LISTEN      3162/kubelet        
tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      5841/kube-proxy     
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      4402/etcd           
tcp        0      0 10.0.2.15:2379          0.0.0.0:*               LISTEN      4402/etcd           
tcp        0      0 10.0.2.15:2380          0.0.0.0:*               LISTEN      4402/etcd           
tcp        0      0 127.0.0.1:10257         0.0.0.0:*               LISTEN      8401/kube-controlle 
tcp        0      0 127.0.0.1:10259         0.0.0.0:*               LISTEN      8332/kube-scheduler 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      939/sshd            
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1117/master         
tcp        0      0 127.0.0.1:58546         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58512         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58502         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58574         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:59130         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:59056         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58524         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58460         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58536         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:59258         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58518         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:59256         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:59020         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58526         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58620         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58600         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58512         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58506         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58534         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58538         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58528         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58602         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58624         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58486         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58470         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:59102         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58504         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58556         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58604         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:59016         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58562         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:59052         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58466         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58596         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58564         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:59050         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:59052         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58478         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:59130         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58494         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58492         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58498         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58490         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58570         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58468         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0     38 127.0.0.1:2379          127.0.0.1:58484         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:59048         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58626         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58508         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58460         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58608         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58530         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:59010         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58524         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58610         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58570         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:59020         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58598         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58614         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58560         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58544         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58540         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:59018         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58534         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58464         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58626         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:59026         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:59044         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58526         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58464         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58490         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:59046         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58594         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:59046         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58624         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58462         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58488         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:59026         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58492         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:59022         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58482         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58504         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58522         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58568         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58544         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58528         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58476         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58622         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58602         ESTABLISHED 4402/etcd           
tcp        0      0 192.168.56.102:22       192.168.56.1:34358      ESTABLISHED 1187/sshd: root@pts 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58564         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58522         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58608         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58562         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:59012         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58506         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58532         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58572         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58598         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58514         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58532         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58450         127.0.0.1:2379          ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58574         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58472         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58530         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:59010         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58614         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58540         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 10.0.2.15:2379          10.0.2.15:36742         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58620         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:59028         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:59014         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58600         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58508         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:59028         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58520         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58484         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58592         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58518         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58482         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58498         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58480         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:59050         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:59048         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58622         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58604         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58502         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58478         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58556         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:59012         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58538         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58468         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58606         ESTABLISHED 4402/etcd           
tcp        0      0 10.0.2.15:36742         10.0.2.15:2379          ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58610         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58488         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58466         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58516         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58476         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58584         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:59056         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:59016         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58584         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58606         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58546         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58566         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:59044         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58594         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58592         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58596         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58496         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58516         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58566         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58486         ESTABLISHED 4402/etcd           
tcp        0      0 10.0.2.15:52074         10.96.0.1:443           ESTABLISHED 7946/storage-provis 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58470         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58462         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58572         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:59018         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58520         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58514         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58560         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:58450         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58536         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:59258         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58480         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:59256         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:58494         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:58472         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:2379          127.0.0.1:59014         ESTABLISHED 4402/etcd           
tcp        0      0 127.0.0.1:59102         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:59022         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58496         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp        0      0 127.0.0.1:58568         127.0.0.1:2379          ESTABLISHED 4299/kube-apiserver 
tcp6       0      0 :::10250                :::*                    LISTEN      3162/kubelet        
tcp6       0      0 :::10251                :::*                    LISTEN      8332/kube-scheduler 
tcp6       0      0 :::10252                :::*                    LISTEN      8401/kube-controlle 
tcp6       0      0 :::10255                :::*                    LISTEN      3162/kubelet        
tcp6       0      0 :::10256                :::*                    LISTEN      5841/kube-proxy     
tcp6       0      0 :::22                   :::*                    LISTEN      939/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1117/master         
tcp6       0      0 :::8443                 :::*                    LISTEN      4299/kube-apiserver 
tcp6       0     34 ::1:47720               ::1:8443                ESTABLISHED 8401/kube-controlle 
tcp6       0      0 ::1:8443                ::1:47706               ESTABLISHED 4299/kube-apiserver 
tcp6       0      0 127.0.0.1:10251         127.0.0.1:50242         TIME_WAIT   -                   
tcp6       0      0 127.0.0.1:10251         127.0.0.1:50232         TIME_WAIT   -                   
tcp6       0      0 127.0.0.1:10251         127.0.0.1:50258         TIME_WAIT   -                   
tcp6       0      0 127.0.0.1:10252         127.0.0.1:49180         TIME_WAIT   -                   
tcp6       0      0 ::1:47744               ::1:8443                ESTABLISHED 8401/kube-controlle 
tcp6       0    475 ::1:8443                ::1:47002               ESTABLISHED 4299/kube-apiserver 
tcp6       0      0 ::1:8443                ::1:47744               ESTABLISHED 4299/kube-apiserver 
tcp6       0      0 ::1:8443                ::1:47450               ESTABLISHED 4299/kube-apiserver 
tcp6       0      0 ::1:47292               ::1:8443                ESTABLISHED 3162/kubelet        
tcp6       0      0 10.0.2.15:8443          10.0.2.15:52074         ESTABLISHED 4299/kube-apiserver 
tcp6       0      0 ::1:8443                ::1:47292               ESTABLISHED 4299/kube-apiserver 
tcp6       0      0 127.0.0.1:10251         127.0.0.1:50250         TIME_WAIT   -                   
tcp6       0      0 127.0.0.1:10252         127.0.0.1:49172         TIME_WAIT   -                   
tcp6       0      0 ::1:47002               ::1:8443                ESTABLISHED 4299/kube-apiserver 
tcp6       0      0 ::1:8443                ::1:47720               ESTABLISHED 4299/kube-apiserver 
tcp6       0      0 127.0.0.1:10252         127.0.0.1:49188         TIME_WAIT   -                   
tcp6       0      0 ::1:47706               ::1:8443                ESTABLISHED 8332/kube-scheduler 
tcp6       0      0 ::1:47450               ::1:8443                ESTABLISHED 5841/kube-proxy     
tcp6       0      0 127.0.0.1:10252         127.0.0.1:49162         TIME_WAIT   -                   
udp        0      0 0.0.0.0:68              0.0.0.0:*                           734/dhclient        
udp        0      0 0.0.0.0:68              0.0.0.0:*                           736/dhclient        
[root@dev01 docker]# minikube dashboard
🔌  Enabling dashboard ...
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
💣  http://127.0.0.1:37998/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/ is not responding properly: Temporary Error: unexpected response code: 404
Temporary Error: unexpected response code: 404
.........
Temporary Error: unexpected response code: 404
[root@dev01 docker]# minikube start --help
Starts a local kubernetes cluster using VM. This command
assumes you have already installed one of the VM drivers: virtualbox/parallels/vmwarefusion/kvm/xhyve/hyperv.

Usage:
  minikube start [flags]

Flags:
      --apiserver-ips ipSlice             A set of apiserver IP Addresses which are used in the generated certificate for kubernetes.  This can be used if you want to make the apiserver available from outside the machine (default [])
      --apiserver-name string             The apiserver name which is used in the generated certificate for kubernetes.  This can be used if you want to make the apiserver available from outside the machine (default "minikubeCA")
      --apiserver-names stringArray       A set of apiserver names which are used in the generated certificate for kubernetes.  This can be used if you want to make the apiserver available from outside the machine
      --apiserver-port int                The apiserver listening port (default 8443)
      --cache-images                      If true, cache docker images for the current bootstrapper and load them into the machine. Always false with --vm-driver=none. (default true)
      --container-runtime string          The container runtime to be used (docker, crio, containerd) (default "docker")
      --cpus int                          Number of CPUs allocated to the minikube VM (default 2)
      --cri-socket string                 The cri socket path to be used
      --disable-driver-mounts             Disables the filesystem mounts provided by the hypervisors (vboxfs, xhyve-9p)
      --disk-size string                  Disk size allocated to the minikube VM (format: <number>[<unit>], where unit = b, k, m or g) (default "20g")
      --dns-domain string                 The cluster dns domain name used in the kubernetes cluster (default "cluster.local")
      --docker-env stringArray            Environment variables to pass to the Docker daemon. (format: key=value)
      --docker-opt stringArray            Specify arbitrary flags to pass to the Docker daemon. (format: key=value)
      --download-only                     If true, only download and cache files for later use - don't install or start anything.
      --enable-default-cni                Enable the default CNI plugin (/etc/cni/net.d/k8s.conf). Used in conjunction with "--network-plugin=cni"
      --extra-config ExtraOption          A set of key=value pairs that describe configuration that may be passed to different components.
                                          		The key should be '.' separated, and the first part before the dot is the component to apply the configuration to.
                                          		Valid components are: kubelet, kubeadm, apiserver, controller-manager, etcd, proxy, scheduler
                                          		Valid kubeadm parameters: ignore-preflight-errors, dry-run, kubeconfig, kubeconfig-dir, node-name, cri-socket, experimental-upload-certs, certificate-key, rootfs, pod-network-cidr
      --feature-gates string              A set of key=value pairs that describe feature gates for alpha/experimental features.
      --gpu                               Enable experimental NVIDIA GPU support in minikube (works only with kvm2 driver on Linux)
  -h, --help                              help for start
      --hidden                            Hide the hypervisor signature from the guest in minikube (works only with kvm2 driver on Linux)
      --host-only-cidr string             The CIDR to be used for the minikube VM (only supported with Virtualbox driver) (default "192.168.99.1/24")
      --hyperkit-vpnkit-sock string       Location of the VPNKit socket used for networking. If empty, disables Hyperkit VPNKitSock, if 'auto' uses Docker for Mac VPNKit connection, otherwise uses the specified VSock.
      --hyperkit-vsock-ports strings      List of guest VSock ports that should be exposed as sockets on the host (Only supported on with hyperkit now).
      --hyperv-virtual-switch string      The hyperv virtual switch name. Defaults to first found. (only supported with HyperV driver)
      --image-mirror-country string       Country code of the image mirror to be used. Leave empty to use the global one. For Chinese mainland users, set it to cn
      --image-repository string           Alternative image repository to pull docker images from. This can be used when you have limited access to gcr.io. Set it to "auto" to let minikube decide one for you. For Chinese mainland users, you may use local gcr.io mirrors such as registry.cn-hangzhou.aliyuncs.com/google_containers (default "registry.cn-hangzhou.aliyuncs.com/google_containers")
      --insecure-registry strings         Insecure Docker registries to pass to the Docker daemon.  The default service CIDR range will automatically be added.
      --iso-url string                    Location of the minikube iso (default "https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.2.0.iso")
      --keep-context                      This will keep the existing kubectl context and will create a minikube context.
      --kubernetes-version string         The kubernetes version that the minikube VM will use (ex: v1.2.3) (default "v1.15.0")
      --kvm-network string                The KVM network name. (only supported with KVM driver) (default "default")
      --memory int                        Amount of RAM allocated to the minikube VM in MB (default 2048)
      --mount                             This will start the mount daemon and automatically mount files into minikube
      --mount-string string               The argument to pass the minikube mount command on start (default "/root:/minikube-host")
      --network-plugin string             The name of the network plugin
      --nfs-share strings                 Local folders to share with Guest via NFS mounts (Only supported on with hyperkit now)
      --nfs-shares-root string            Where to root the NFS Shares (defaults to /nfsshares, only supported with hyperkit now) (default "/nfsshares")
      --no-vtx-check                      Disable checking for the availability of hardware virtualization before the vm is started (virtualbox)
      --registry-mirror strings           Registry mirrors to pass to the Docker daemon
      --service-cluster-ip-range string   The CIDR to be used for service cluster IPs. (default "10.96.0.0/12")
      --uuid string                       Provide VM UUID to restore MAC address (only supported with Hyperkit driver).
      --vm-driver string                  VM driver is one of: [virtualbox parallels vmwarefusion kvm xhyve hyperv hyperkit kvm2 vmware none] (default "virtualbox")
      --xhyve-disk-driver string          The disk driver to use [ahci-hd|virtio-blk] (only supported with xhyve driver) (default "ahci-hd")

Global Flags:
      --alsologtostderr                  log to standard error as well as files
  -b, --bootstrapper string              The name of the cluster bootstrapper that will set up the kubernetes cluster. (default "kubeadm")
      --log_backtrace_at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log_dir string                   If non-empty, write log files in this directory
      --logtostderr                      log to standard error instead of files
  -p, --profile string                   The name of the minikube VM being used.  
                                         	This can be modified to allow for multiple minikube instances to be run independently (default "minikube")
      --stderrthreshold severity         logs at or above this threshold go to stderr (default 2)
  -v, --v Level                          log level for V logs
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging
[root@dev01 docker]# kubectl get pod
No resources found.
[root@dev01 docker]# minikube --help
Minikube is a CLI tool that provisions and manages single-node Kubernetes clusters optimized for development workflows.

Usage:
  minikube [command]

Available Commands:
  addons         Modify minikube's kubernetes addons
  cache          Add or delete an image from the local cache.
  completion     Outputs minikube shell completion for the given shell (bash or zsh)
  config         Modify minikube config
  dashboard      Access the kubernetes dashboard running within the minikube cluster
  delete         Deletes a local kubernetes cluster
  docker-env     Sets up docker env variables; similar to '$(docker-machine env)'
  help           Help about any command
  ip             Retrieves the IP address of the running cluster
  kubectl        Run kubectl
  logs           Gets the logs of the running instance, used for debugging minikube, not user code
  mount          Mounts the specified directory into minikube
  profile        Profile gets or sets the current minikube profile
  service        Gets the kubernetes URL(s) for the specified service in your local cluster
  ssh            Log into or run a command on a machine with SSH; similar to 'docker-machine ssh'
  ssh-key        Retrieve the ssh identity key path of the specified cluster
  start          Starts a local kubernetes cluster
  status         Gets the status of a local kubernetes cluster
  stop           Stops a running local kubernetes cluster
  tunnel         tunnel makes services of type LoadBalancer accessible on localhost
  update-check   Print current and latest version number
  update-context Verify the IP address of the running cluster in kubeconfig.
  version        Print the version of minikube

Flags:
      --alsologtostderr                  log to standard error as well as files
  -b, --bootstrapper string              The name of the cluster bootstrapper that will set up the kubernetes cluster. (default "kubeadm")
  -h, --help                             help for minikube
      --log_backtrace_at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log_dir string                   If non-empty, write log files in this directory
      --logtostderr                      log to standard error instead of files
  -p, --profile string                   The name of the minikube VM being used.  
                                         	This can be modified to allow for multiple minikube instances to be run independently (default "minikube")
      --stderrthreshold severity         logs at or above this threshold go to stderr (default 2)
  -v, --v Level                          log level for V logs
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging

Use "minikube [command] --help" for more information about a command.
[root@dev01 docker]# minikube stop --help
Stops a local kubernetes cluster running in Virtualbox. This command stops the VM
itself, leaving all files intact. The cluster can be started again with the "start" command.

Usage:
  minikube stop [flags]

Flags:
  -h, --help   help for stop

Global Flags:
      --alsologtostderr                  log to standard error as well as files
  -b, --bootstrapper string              The name of the cluster bootstrapper that will set up the kubernetes cluster. (default "kubeadm")
      --log_backtrace_at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log_dir string                   If non-empty, write log files in this directory
      --logtostderr                      log to standard error instead of files
  -p, --profile string                   The name of the minikube VM being used.  
                                         	This can be modified to allow for multiple minikube instances to be run independently (default "minikube")
      --stderrthreshold severity         logs at or above this threshold go to stderr (default 2)
  -v, --v Level                          log level for V logs
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging
[root@dev01 docker]# minikube stop 
✋  Stopping "minikube" in none ...
🛑  "minikube" stopped.
[root@dev01 docker]# 

## 2 关于docker和minikube启动参数的说明

### 2.1 registry mirror

文档参考链接：https://docs.docker.com/registry/recipes/mirror/

根据[registry文档](https://docs.docker.com/registry/)的描述：The Registry is a stateless, highly scalable server side application that stores and lets you distribute Docker images. The Registry is open-source, under the permissive [Apache license](http://en.wikipedia.org/wiki/Apache_License).

registry主要是用来存储和分发镜像

registry mirror主要是用作缓存镜像，相当于代理，文档描述如下：

# Registry as a pull through cache

Estimated reading time: 4 minutes

## Use-case

If you have multiple instances of Docker running in your environment, such as multiple physical or virtual machines all running Docker, each daemon goes out to the internet and fetches an image it doesn’t have locally, from the Docker repository. You can run a local registry mirror and point all your daemons there, to avoid this extra internet traffic.

### Alternatives

Alternatively, if the set of images you are using is well delimited, you can simply pull them manually and push them to a simple, local, private registry.

Furthermore, if your images are all built in-house, not using the Hub at all and relying entirely on your local registry is the simplest scenario.

### Gotcha

It’s currently not possible to mirror another private registry. Only the central Hub can be mirrored.

### Solution

The Registry can be configured as a pull through cache. In this mode a Registry responds to all normal docker pull requests but stores all content locally.

## How does it work?

The first time you request an image from your local registry mirror, it pulls the image from the public Docker registry and stores it locally before handing it back to you. On subsequent requests, the local registry mirror is able to serve the image from its own storage.

### What if the content changes on the Hub?

When a pull is attempted with a tag, the Registry checks the remote to ensure if it has the latest version of the requested content. Otherwise, it fetches and caches the latest content.

### What about my disk?

In environments with high churn rates, stale data can build up in the cache. When running as a pull through cache the Registry periodically removes old content to save disk space. Subsequent requests for removed content causes a remote fetch and local re-caching.

To ensure best performance and guarantee correctness the Registry cache should be configured to use the `filesystem` driver for storage.

## Run a Registry as a pull-through cache

The easiest way to run a registry as a pull through cache is to run the official Registry image. At least, you need to specify `proxy.remoteurl` within `/etc/docker/registry/config.yml` as described in the following subsection.

Multiple registry caches can be deployed over the same back-end. A single registry cache ensures that concurrent requests do not pull duplicate data, but this property does not hold true for a registry cache cluster.

### Configure the cache

To configure a Registry to run as a pull through cache, the addition of a `proxy` section is required to the config file.

To access private images on the Docker Hub, a username and password can be supplied.

```
proxy:
  remoteurl: https://registry-1.docker.io
  username: [username]
  password: [password]
```

> **Warning**: If you specify a username and password, it’s very important to understand that private resources that this user has access to Docker Hub is made available on your mirror. **You must secure your mirror** by implementing authentication if you expect these resources to stay private!

> **Warning**: For the scheduler to clean up old entries, `delete` must be enabled in the registry configuration. See [Registry Configuration](https://docs.docker.com/registry/configuration/) for more details.

### Configure the Docker daemon

Either pass the `--registry-mirror` option when starting `dockerd` manually, or edit [`/etc/docker/daemon.json`](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file) and add the `registry-mirrors` key and value, to make the change persistent.

```
{
  "registry-mirrors": ["https://<my-docker-mirror-host>"]
}
```

Save the file and reload Docker for the change to take effect.

> Some log messages that appear to be errors are actually informational messages.
>
> Check the `level` field to determine whether the message is warning you about an error or is giving you information. For example, this log message is informational:
>
> ```
> time="2017-06-02T15:47:37Z" level=info msg="error statting local store, serving from upstream: unknown blob" go.version=go1.7.4
> ```
>
> It’s telling you that the file doesn’t exist yet in the local cache and is being pulled from upstream.

## Use case: the China registry mirror

The URL of the registry mirror for China is `registry.docker-cn.com`. You can pull images from this mirror just like you do for other registries by specifying the full path, including the registry, in your `docker pull` command, for example:

```
$ docker pull registry.docker-cn.com/library/ubuntu
```

You can add `"https://registry.docker-cn.com"` to the `registry-mirrors` array in [`/etc/docker/daemon.json`](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file) to pull from the China registry mirror by default.

```
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

Save the file and reload Docker for the change to take effect.

Or, you can configure the Docker daemon with the `--registry-mirror` startup parameter:

```
$ dockerd --registry-mirror=https://registry.docker-cn.com
```

### 2.2 insecure registy

文档参考链接：https://docs.docker.com/engine/reference/commandline/dockerd/#insecure-registries

insecure registry主要是用来配置docker daemon信任一个不安全的仓库，从而能从仓库中拉取镜像，文档描述如下：

### Insecure registries

Docker considers a private registry either secure or insecure. In the rest of this section, *registry* is used for *private registry*, and `myregistry:5000` is a placeholder example for a private registry.

A secure registry uses TLS and a copy of its CA certificate is placed on the Docker host at `/etc/docker/certs.d/myregistry:5000/ca.crt`. An insecure registry is either not using TLS (i.e., listening on plain text HTTP), or is using TLS with a CA certificate not known by the Docker daemon. The latter can happen when the certificate was not found under `/etc/docker/certs.d/myregistry:5000/`, or if the certificate verification failed (i.e., wrong CA).

By default, Docker assumes all, but local (see local registries below), registries are secure. Communicating with an insecure registry is not possible if Docker assumes that registry is secure. In order to communicate with an insecure registry, the Docker daemon requires `--insecure-registry` in one of the following two forms:

- `--insecure-registry myregistry:5000` tells the Docker daemon that myregistry:5000 should be considered insecure.
- `--insecure-registry 10.1.0.0/16` tells the Docker daemon that all registries whose domain resolve to an IP address is part of the subnet described by the CIDR syntax, should be considered insecure.

The flag can be used multiple times to allow multiple registries to be marked as insecure.

If an insecure registry is not marked as insecure, `docker pull`, `docker push`, and `docker search` will result in an error message prompting the user to either secure or pass the `--insecure-registry` flag to the Docker daemon as described above.

Local registries, whose IP address falls in the 127.0.0.0/8 range, are automatically marked as insecure as of Docker 1.3.2. It is not recommended to rely on this, as it may change in the future.

Enabling `--insecure-registry`, i.e., allowing un-encrypted and/or untrusted communication, can be useful when running a local registry. However, because its use creates security vulnerabilities it should ONLY be enabled for testing purposes. For increased security, users should add their CA to their system’s list of trusted CAs instead of enabling `--insecure-registry`.

#### LEGACY REGISTRIES

Starting with Docker 17.12, operations against registries supporting only the legacy v1 protocol are no longer supported. Specifically, the daemon will not attempt `push`, `pull` and `login` to v1 registries. The exception to this is `search` which can still be performed on v1 registries.

The `disable-legacy-registry` configuration option has been removed and, when used, will produce an error on daemon startup.

### 2.3 关于registry的设置

文档参考链接：https://docs.docker.com/registry/configuration/



