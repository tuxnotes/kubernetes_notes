# 1 系统初始化

用于测试和开发环境，生产环境参考[文档](https://github.com/k8sre/docs/blob/master/kubernetes/kubernetes%E9%AB%98%E5%8F%AF%E7%94%A8%E9%9B%86%E7%BE%A4%E4%B9%8B%E4%BA%8C%E8%BF%9B%E5%88%B6%E9%83%A8%E7%BD%B2.md)

所有数据保存在etcd中。master节点尽量使用ssd磁盘作为etcd数据目录、

haproxy作为apiserver 4层代理，可根据需要更爱为其他方式

## 1.1 前提条件

- 所有机器可以访问互联网，并且内网互通
- 使用root用户进行操作

### 1.2 配置密钥登录

在master-01上生成密钥并拷贝到其他节点

```bash
# ssh-keygen -t rsa -b 4096 -C kubernetes
# ssh-copy-id root@master-01
```

## 1.3 修改主机名

```bash
# hostnamectl set-hostname master-01
```

## 1.4 修改软件源

```bash
# curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
# curl -o /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo
# yum clean all && yum makecache fast
```

## 1.5 关闭swap

删除/etc/fstab中swap配置，并执行以下命令：

```bash
# swapoff -a
# sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab # (gitee)
```

## 1.6关闭防火墙

```bash
# systemctl stop iptables.service
# systemctl disable iptables.service
# systemctl stop firewalld.service
# systemctl disable firewalld.service
#iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat && iptables -P FORWARD ACCEPT # (gitee)

# 关闭dnsmasq(否则可能导致docker容器无法解析域名)
# service dnsmasq stop && systemctl disable dnsmasq # (gitee)
```

## 1.7 关闭selinux

```bash
# setenforce 0
# sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
```

## 1.8 配置chrony时间服务

```bash
# yum -y install chrony
# cat > /etc/chrony.conf << EOF
server ntp.cloud.aliyuncs.com iburst
server ntp.aliyun.com iburst
startumweight 0
driftfile /var/lib/chrony/drift
rtcsync
makestep 10 3
bindcmdaddress 127.0.0.1
bindcmdaddress ::1
keyfile /etc/chrony.keys
commandkey 1
generatecommandkey
logchange 0.5
logdir /var/log/chrony
EOF
# systemctl enable chronyd
# systemctl restart chronyd
# chronyc tracking
```

虚拟机可以在同步下系统及硬件时钟

```bash
# clock --hctosys
```

## 1.9 安装依赖

```bash
#yum -y install vim wget lrzsz telnet nmap-ncat make net-tools gcc gcc-c++ cmake bash-completion mtr python-devel sshpass conntrack ipvsadm ipset jq libnetfilter_conntrack-devel libnetfilter_conntrack conntrack-tools

# yum install -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp  # (gitee)
```

## 1.10 加载内核模块

```bash
# modprobe ip_vs
# modprobe ip_vs_rr
# modprobe ip_vs_wrr
# modprobe ip_vs_sh
# modprobe br_netfilter
# modprobe nf_conntrack_ipv4
```

## 1.11 内核优化

```bash
# cat > /etc/sysctl.conf << EOF
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
fs.file-max=655360
net.ipv4.tcp_tw_recycle=0
vm.swappiness=0 # 禁止使用 swap 空间，只有当系统 OOM 时才允许使用它
vm.overcommit_memory=1 # 不检查物理内存是否够用
vm.panic_on_oom=0 # 开启 OOM
fs.inotify.max_user_instances=8192
fs.inotify.max_user_watches=1048576
fs.file-max=52706963
fs.nr_open=52706963
net.ipv6.conf.all.disable_ipv6=1
net.netfilter.nf_conntrack_max=2310720
net.ipv4.ip_local_port_range=20000 60999
EOF

# 生效配置文件  (gitee)
# sysctl -p /etc/sysctl.d/kubernetes.conf
```

## 1.12 配置journald

```bash
# mkdir /var/log/journal # 持久化保存日志的目录
# mkdir /etc/systemd/journald.conf.d
# cat > /etc/systemd/journald.conf.d/99-prophet.conf <<EOF
[Journal]
# 持久化保存到磁盘
Storage=persistent
# 压缩历史日志
Compress=yes
SyncIntervalSec=5m
RateLimitInterval=30s
RateLimitBurst=1000
# 最大占用空间 10G
SystemMaxUse=10G
# 单日志文件最大 200M
SystemMaxFileSize=200M
# 日志保存时间 2 周
MaxRetentionSec=2week
# 不将日志转发到 syslog
ForwardToSyslog=no
EOF
# systemctl restart systemd-journald
```

## 1.13 挂在数据盘

```bash
# parted /dev/sdb mklabel gpt
# parted /dev/sdb "mkpart primary xfs 0 -0"
# mkfs.xfs  /dev/sdb1
# mount /dev/sdb1 /data
# echo "/dev/sdb1 /data    xfs     defaults        0 0" >> /etc/fstab 
```

## 1.14创建证书目录

在k8s所有节点创建

```bash
# mkdir -p /etc/kubernetes/ssl/
```

## 1.15分发kubernetes二进制文件

```bash
# wget https://dl.k8s.io/v1.15.4/kubernetes-server-linux-amd64.tar.gz
# tar zxvf kubernetes-server-linux-amd64.tar.gz
# cd kubernetes/server/bin
# scp {kube-apiserver,kube-controller-manager,kube-scheduler,kubectl} ${MASTER}:/usr/bin/
# scp {kubelet,kube-proxy} ${NODE}:/usr/bin/
# ssh ${IP_ADDR} "chmod +x /usr/bin/kube*"
```

## 1.16 添加kuberbetes运行用户

master节点上执行

```bash
# useradd -u 200 -d /var/kube -s /sbin/nologin kube
```

# 2 签发证书

## 2.1 签发ca证书

### 2.1.1 准备ca.cnf

```bahs
# cat > ca.cnf << EOF
[ req ]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]

[ v3_req ]
keyUsage = critical, cRLSign, keyCertsing, digitalSignature, keyEncipherment
extendedkeyUsage = serverAuth, clientAuth
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:2
EOF
```

### 2.1.2 生成ca.key

```bash
# openssl genrsa -out ca.key 4096
```

### 2.1.3 签发ca

```bash
# openssl req -x509 -new -nodes -key ca.key -days 1825 -out ca.pem -subj \
			"/CN=kubernetes/OU=System/C=CN/ST=Shanghai/L=Shanghai/O=k8s" \
			-config ca.cnf -extensions v3_req
```

- CN: Common Name, kube-apiserver从证书中提取该字段作为请求的用户名(User Name), 浏览器使用该字段验证网站是否合法
- O: Organization, kube-apiserver从证书中提取该字段做为请求用户所属的组(Group)
- kube-apiserver将提取的User, Group作为RBAC授权的用户标识

### 2.1.4 校验证书

```bash
# openssl x509 -noout -text -in ca.pem
```

##  2.2 签发etcd证书

### 2.2.1 准备etcd.cnf

```bash
# cat > etcd.cnf << EOF
[ req ]
req_extensioins = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
IP.1 = 127.0.0.1
IP.2 = 10.15.1.201
IP.2 = 10.15.1.202
IP.2 = 10.15.1.203
```

- ip 对应etcd节点的ip地址

### 2.2.2 生成key

```bash
# openssl genrsa -out etcd.key 4096
```

### 2.2.3 生成证书请求

```bash
# openssl req -new -key etcd.key -out etcd.csr \
			-subj "/CN=etcd/OU=System/C=CN/ST=Shanghai/L=Shanghai/O=k8s" \
			-config etcd.cnf
```

### 2.2.4 签发证书

```bash
# openssl x509 -req -in etcd.csr \
			-CA ca.pem -CAkey ca.key -CAcreateserial \
			-out etcd.pem -days 1825 \
			-extfile etcd.cnf -extensions v3_req
```

### 2.2.5 校验证书

```bash
# openssl x509 -noout -text -in etcd.pem
```





