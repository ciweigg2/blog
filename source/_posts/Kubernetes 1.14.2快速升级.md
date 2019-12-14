cover: http://ciwei2.cn-sh2.ufileos.com/63.jpg
title: Kubernetes 1.14.2快速升级
date: 2019-06-01 14:57:00
tags: [kubernetes]
categories: [综合]
---
1、升级kubeadm/kubectl/kubelet版本

```java
sudo apt install kubeadm=1.14.2-00 kubectl=1.14.2-00 kubelet=1.14.2-00
```

<!--more-->

查看该版本的容器镜像版本：

```java
kubeadm config images list
```

输出如下：

~# kubeadm config images list

```java
k8s.gcr.io/kube-apiserver:v1.14.2
k8s.gcr.io/kube-controller-manager:v1.14.2
k8s.gcr.io/kube-scheduler:v1.14.2
k8s.gcr.io/kube-proxy:v1.14.2
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.3.10
k8s.gcr.io/coredns:1.3.1
```

2、拉取容器镜像

原始的kubernetes镜像文件在gcr上，不能直接下载。我给镜像到了阿里云的杭州机房的容器仓库里，拉取还是比较快的

vi init.sh

```java
echo ""
echo "=========================================================="
echo "Pull Kubernetes v1.14.2 Images from aliyuncs.com ......"
echo "=========================================================="
echo ""

MY_REGISTRY=registry.cn-hangzhou.aliyuncs.com/openthings

## 拉取镜像
docker pull ${MY_REGISTRY}/k8s-gcr-io-kube-apiserver:v1.14.2
docker pull ${MY_REGISTRY}/k8s-gcr-io-kube-controller-manager:v1.14.2
docker pull ${MY_REGISTRY}/k8s-gcr-io-kube-scheduler:v1.14.2
docker pull ${MY_REGISTRY}/k8s-gcr-io-kube-proxy:v1.14.2
docker pull ${MY_REGISTRY}/k8s-gcr-io-etcd:3.3.10
docker pull ${MY_REGISTRY}/k8s-gcr-io-pause:3.1
docker pull ${MY_REGISTRY}/k8s-gcr-io-coredns:1.3.1


## 添加Tag
docker tag ${MY_REGISTRY}/k8s-gcr-io-kube-apiserver:v1.14.2 k8s.gcr.io/kube-apiserver:v1.14.2
docker tag ${MY_REGISTRY}/k8s-gcr-io-kube-scheduler:v1.14.2 k8s.gcr.io/kube-scheduler:v1.14.2
docker tag ${MY_REGISTRY}/k8s-gcr-io-kube-controller-manager:v1.14.1 k8s.gcr.io/kube-controller-manager:v1.14.2
docker tag ${MY_REGISTRY}/k8s-gcr-io-kube-proxy:v1.14.2 k8s.gcr.io/kube-proxy:v1.14.2
docker tag ${MY_REGISTRY}/k8s-gcr-io-etcd:3.3.10 k8s.gcr.io/etcd:3.3.10
docker tag ${MY_REGISTRY}/k8s-gcr-io-pause:3.1 k8s.gcr.io/pause:3.1
docker tag ${MY_REGISTRY}/k8s-gcr-io-coredns:1.3.1 k8s.gcr.io/coredns:1.3.1

echo ""
echo "=========================================================="
echo "Pull Kubernetes v1.14.2 Images FINISHED."
echo "into registry.cn-hangzhou.aliyuncs.com/openthings, "
echo "           by openthings@https://my.oschina.net/u/2306127."
echo "=========================================================="

echo ""
```

保存为shell脚本，然后执行。

或者，下载脚本：https://github.com/openthings/kubernetes-tools/blob/master/kubeadm/2-images/

3、升级Kubernetes集群

全新安装：

```java
#指定IP地址，1.14.2版本(--apiserver-advertise-address=10.1.1.199不需要了呀)：
sudo kubeadm init --kubernetes-version=v1.14.2 --pod-network-cidr=10.244.0.0/16

#注意，CoreDNS已经内置，不再需要参数--feature-gates CoreDNS=true
```

先查看一下需要升级的各个组件的版本

使用kubeadm upgrade plan ，输出的版本升级信息如下：

```java
COMPONENT            CURRENT   AVAILABLE
API Server           v1.14.1   v1.14.2
Controller Manager   v1.14.1   v1.14.2
Scheduler            v1.14.1   v1.14.2
Kube Proxy           v1.14.1   v1.14.2
CoreDNS              1.3.1     1.3.1
Etcd                 3.3.10    3.3.10
```

确保上面的容器镜像已经下载（如果没有提前下载，可能被网络阻隔导致挂起），然后执行升级：

```java
kubeadm upgrade -y apply v1.14.2
```

看到下面信息，就OK了

```java
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.14.2". Enjoy!
```

然后，配置当前用户环境：

```java
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

就可以使用 kubectl version 来查看状态和 kubectl cluster-info 查看服务地址

4、工作节点的升级

每个工作节点需要拉取上面对应版本的镜像，以及安装kubelet的对应版本

检查版本：

```java
~$ kubectl version
```

查看Pod信息：

```java
kubectl get pod --all-namespaces
```

完成

5、其它：

出现整个集群无法访问，kubectl get node失败，kubectl version时apiserver访问失败

查看其中一个节点route，再次出现神秘的podsxx 255.255.255.255路由记录，route del删除记录失败

运行sudo netplan apply后，路由记录消失，节点恢复可访问