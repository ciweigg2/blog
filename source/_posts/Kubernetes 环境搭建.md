cover: http://ciwei2.cn-sh2.ufileos.com/67.jpg
title: Kubernetes(k8s) 环境搭建
date: 2018-09-09 13:47:58
tags: [kubernetes]
categories: [综合]
---
### 环境
三台 CentOS 7.4 服务器：kube1 、kube2 、kube3 ，配置：2 核 4G
<!--more-->

关闭、禁用防火墙：

```java
systemctl stop firewalld

systemctl disable firewalld
```

禁用SELINUX：

```java
setenforce 0
```

创建 /etc/sysctl.d/k8s.conf 文件，添加如下内容：
```java
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```

执行如下命令使修改生效：
```java
modprobe br_netfilter

sysctl -p /etc/sysctl.d/k8s.conf
```

### 安装 Docker (18.0.3-ce测试的版本)
```java
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# Step 2: 添加软件源信息
yum-config-manager \
  --add-repo \
  https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# Step 3: 更新并安装 Docker-CE
yum install docker-ce-18.03.0.ce

# Step 4: 设置开机启动
sudo systemctl enable docker

# Step 5: 开启Docker服务
sudo service docker start
```

配置阿里云镜像加速器：
```java
mkdir -p /etc/docker

tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://obww7jh1.mirror.aliyuncs.com"]
}
EOF

systemctl daemon-reload

systemctl restart docker
```

Pull 必须的镜像 ( Master 节点 kube1 ):

```java
export image=pause:3.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/${image}
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/${image} k8s.gcr.io/${image}
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/${image}

export image=kube-apiserver-amd64:v1.11.2
docker pull registry.cn-hangzhou.aliyucs.com/google_containers/${image}
docker tag registry.cn-hangzhoou.aliyuncs.com/google_containers/${image} k8s.gcr.io/${image}
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/${image}

export image=kube-scheduler-amd64:v1.11.2
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/${image}
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/${image} k8s.gcr.io/${image}
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/${image}

export image=kube-controller-manager-amd64:v1.11.2
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/${image}
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/${image} k8s.gcr.io/${image}
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/${image}

export image=kue-proxy-amd64:v1.11.2
docker pull registry.cn-hangzhou.aliyuncs..com/google_containers/${image}
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/${image} k8s.gcr.io/${image}
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/${image}

export image=k8s-dns-kube-dns-amd64:1.14.8
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/${image}
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/${image} k8s.gcr.io/${image}
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/${image}

export image=k8s-dns-dnsmasq-nanny-amd64:1.14.8
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/${image}
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/${image} k8s.gcr.io/${image}
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/${image}

export image=k8s-dns-sidecar-amd64:1.14.8
docker pull registry.cn-hangzhou.aliyuncs.com/google_containerll registry.cn-hangzhou.aliyuncs.com/google_containers/${image}
ontainers/${image} k8s.gcr.io/${image}
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/${image}

export image=etcd-amd64:3.2.18
dhou.aliyuncs.com/google_containers/${image}

export image=coredns:1.1.3
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/${image}
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/${image} k8s.gcr.io/${image}
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/${image}
```

Node 节点 kube2 / kube3
```java
export image=pause:3.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/${image}
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/${image} k8s.gcr.io/${image}
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/${image}

export image=kube-proxy-amd64:v1.11.2
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/${image}
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/${image} k8s.gcr.io/${image}
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/${image}
```

Master和node 节点 

安装 kubelet kubeadm kubectl
```java
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubelet kubeadm kubectl

systemctl enable kubelet && systemctl start kubelet
```

配置 kubelet

安装完成后，我们还需要对 kubelet 进行配置，因为用 yum 源的方式安装的 kubelet 生成的配置文件将参数 --cgroup-driver 改成了 systemd，而 docker 的 cgroup-driver 是 cgroupfs，这二者必须一致才行，我们可以通过 docker info 命令查看：

```java
$ docker info | grep Cgroup
Cgroup Driver: cgroupfs
```

修改文件 kubelet 的配置文件 /etc/systemd/system/kubelet.service.d/10-kubeadm.conf ，将其中的 KUBELET_CGROUP_ARGS 参数更改成 cgroupfs，
没有的话则添加

```java
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs"
```

另外还有一个问题是关于交换分区的，Kubernetes 从 1.8 开始要求关闭系统的 Swap ，如果不关闭，默认配置的 kubelet 将无法启动：

```java
swapoff -a
```

修改完成后，重新加载我们的配置文件即可：

```java
systemctl daemon-reload
```

### 构建 Kubernetes 集群

如果初始化集群报错了说明版本下载的没有对应，重新下载镜像并且打tag(镜像都在国外的，所以必须ali下载下来打上tag才能拉下来)

```java
查看1.11.2版本的k8s可以使用命令获取需要的镜像版本如下（版本一定要对应）:
kubeadm config images list

k8s.gcr.io/kube-apiserver-amd64:v1.11.2
k8s.gcr.io/kube-controller-manager-amd64:v1.11.2
k8s.gcr.io/kube-scheduler-amd64:v1.11.2
k8s.gcr.io/kube-proxy-amd64:v1.11.2
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd-amd64:3.2.18
k8s.gcr.io/coredns:1.1.3
```

1、初始化 Master 节点 kube1

```java
kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=172.17.58.201 --kubernetes-version=v1.11.2
```

* --pod-network-cidr ：后续安装 flannel 的前提条件，且值为 10.244.0.0/16， [参考资料](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
* --apiserver-advertise-address ：Master 节点的 IP 地址

输出日志：

```java
.....
[addons] Applied essential addon: kube-dns
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 172.17.58.201:6443 --token 831rfg.dw0vyb1h3beab5as --discovery-token-ca-cert-hash sha256:623681fde5b2bf564a8631942f31797f9bef75f40b14a86ef75e1d31b43709f1
```

从日志中，可以看出，要使用集群，需要执行如下命令：

```java
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

还需要部署一个 Pod Network 到集群中，此处选择 flannel ：

```java
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml
```

至此，Master 节点初始化完毕，查看集群相关信息：

```java
# 查看集群信息
$ kubectl cluster-info
Kubernetes master is running at https://172.17.58.201:6443
KubeDNS is running at https://172.17.58.201:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

```java
# 查看节点信息
$ kubectl get nodes
NAME           STATUS    ROLES     AGE       VERSION
lab-backend1   Ready     master    1m        v1.10.4
```

```java
# 查看 Pods 信息
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                   READY     STATUS    RESTARTS   AGE
kube-system   etcd-lab-backend1                      1/1       Running   0          2m
kube-system   kube-apiserver-lab-backend1            1/1       Running   0          1m
kube-system   kube-controller-manager-lab-backend1   1/1       Running   0          2m
kube-system   kube-dns-86f4d74b45-zbnz6              3/3       Running   0          3m
kube-system   kube-flannel-ds-n67zn                  1/1       Running   0          1m
kube-system   kube-proxy-qdmqq                       1/1       Running   0          3m
kube-system   kube-scheduler-lab-backend1            1/1       Running   0          2m
```

如果初始化过程出现问题，使用如下命令重置：

```java
kubeadm reset

rm -rf /var/lib/cni/

rm -f $HOME/.kube/config
```

2、添加 Worker 节点

方式 ① 使用 kubeadm init 时返回的信息kube3

```java
kubeadm join 172.17.58.201:6443 --token 831831rf83188831rfg.dw0vyb1h3beab5as --discovery-token-ca-cert-hash sha256:623681fde5b2bf564a8631942f31797f9bef75f40b14a86ef75e1d31b43709f1
```

方式 ② 重新生成 token kube1

```java
kubeadm token generate

kubeadm token create <generated-token> --print-join-command --ttl=24h
```

> --ttl=24h 代表这个Token 的有效期为 24 小时，初始化默认生成的 token 有效期也为 24 小时

加入集群 kube2 / kube3

```java
kubeadm join 172.17.58.201:6443 --token 41ts3r.n2vw06xbniouo6u5 --discovery-token-ca-cert-hash sha256:f958e234e8554c2352127f356a7eb7dad422c10df9a749156df36e5972cba38b
```

再次查看集群节点 kube1

```java
$ kubectl get nodes
NAME           STATUS    ROLES     AGE       VERSION
lab-backend1   Ready     master    6m        v1.11.2
lab-backend2   Ready     <none>    56s       v1.11.2
lab-backend3   Ready     <none>    14s       v1.11.2
```

至此，1 Master + 2 Worker 的 kubernetes 集群就创建成功了
