title: kind安装测试环境集群(不能用于生产)
date: 2019-09-20 18:09:41
tags: [kubernetes,kind]
categories: [综合]
---
Kind（Kubernetes in Docker） 是一个 Kubernetes 孵化项目，Kind 是一套开箱即用的 Kubernetes 环境搭建方案。顾名思义，就是将 Kubernetes 所需要的所有组件，全部部署在一个 Docker 容器中，可以很方便的搭建 Kubernetes 集群

<!--more-->

> 项目地址：https://github.com/kubernetes-sigs/kind

### Kind 可以做什么？

1. 快速创建一个或多个 Kubernetes 集群
2. 支持部署高可用的 Kubernetes 集群
3. 支持从源码构建并部署一个 Kubernetes 集群
4. 可以快速低成本体验一个最新的 Kubernetes 集群，并支持 Kubernetes 的绝大部分功能
5. 支持本地离线运行一个多节点集群

### Kind 有哪些优势？

1. 最小的安装依赖，仅需要安装 Docker 即可
2. 使用方法简单，只需 Kind Cli 工具即可快速创建集群
3. 使用容器来模似 Kubernetes 节点
4. 内部使用 Kubeadm 的官方主流部署工具
5. 通过了 CNCF 官方的 K8S Conformance 测试

### Kind 是如何工作的?

![](/images/kind.gif)

安装 Kind

------

安装完成之后，我们可以来看看 Kind 支持哪些命令行操作。

```yaml
$ kind
kind creates and manages local Kubernetes clusters using Docker container 'nodes'

Usage:
  kind [command]

Available Commands:
  build       Build one of [base-image, node-image]
  create      Creates one of [cluster]
  delete      Deletes one of [cluster]
  export      exports one of [logs]
  get         Gets one of [clusters, nodes, kubeconfig-path]
  help        Help about any command
  load        Loads images into nodes
  version     prints the kind CLI version

Flags:
  -h, --help              help for kind
      --loglevel string   logrus log level [panic, fatal, error, warning, info, debug] (default "warning")
      --version           version for kind

Use "kind [command] --help" for more information about a command.
```

> 简单说下几个比较常用选项的含义：

- build：用来从 Kubernetes 源代码构建一个新的镜像。
- create：创建一个 Kubernetes 集群。
- delete：删除一个 Kubernetes 集群。
- get：可用来查看当前集群、节点信息以及 Kubectl 配置文件的地址。
- load：从宿主机向 Kubernetes 节点内导入镜像。

使用 Kind 创建 Kubernetes 集群

------

### 搭建一个单节点集群

搭建单节点集群是 Kind 最基础的功能，当然使用起来也很简单，仅需一条指令即可完成。

```yaml
$ kind create cluster --name my-cluster
Creating cluster "my-cluster" ...
 ✓ Ensuring node image (kindest/node:v1.15.3) 

                                                ✓ Preparing nodes 

                                                                    ✓ Creating kubeadm config 

                                                                                                ✓ Starting control-plane ️ 
Cluster creation complete. You can now use the cluster with:

exFIG="$(kind get kubeconfig-path --name="my-cluster")"
ku"
ku"
ku"
kubectl cluster-info
```

> 以上命令中 --name 是可选参数。如果不指定，默认创建出来的集群名字为 kind。

使用默认安装的方式时，我们没有指定任何配置文件。从安装过程的输出来看，一共分为 4 步：

- 检查本地环境是否存在一个基础的安装镜像，默认是 kindest/node:v1.15.3，该镜像里面包含了所有需要安装的东西，包括：kubectl、kubeadm、kubelet 的二进制文件，以及安装对应版本 Kubernetes 所需要的镜像。
- 准备 Kubernetes 节点，主要就是启动容器、解压镜像这类的操作。
- 建立对应的 kubeadm 的配置，完成之后就通过 kubeadm 进行安装。安装完成后还会做一些清理操作，比如：删掉主节点上的污点，否则对于没有容忍的 Pod 无法完成部署。
- 上面所有操作都完成后，就成功启动了一个 Kubernetes 集群并输出一些操作集群的提示信息。

> 默认情况下，Kind 会先下载 kindest/node:v1.15.3 镜像。如果你想指定不同版本，可以使用 --image 参数，类似这样：kind create cluster --image kindest/node:v1.15.3 
>
> kindest/node 这个镜像目前托管于 Docker Hub 上，下载时可能会较慢。同样的问题 Kind 进行集群的创建也是存在的，Kind 实际使用 Kubeadm 进行集群的创建。对 Kubeadm 有所了解的同学都知道它默认使用的镜像在国内是不能访问的，所以一样需要自行解决网络问题。
>
> 如果你存在上面说的网络问题，最好配置一个国内的加速器或者镜像源。如果你还不知道如何配置加速器和镜像源可以参考：「Docker / Kubernetes 镜像源不可用，教你几招搞定它！」和 「 Docker 下使用 DaoCloud / 阿里云镜像加速」两篇文章。

接下来，我们根据上面命令执行完后，输出的提示信息进行操作来验证一下集群是否部署成功。

```
# 获取指定集群的配置文件所在的路径

$ export KUBECONFIG="$(kind get kubeconfig-path --name="my-cluster")"
$ kubectl cluster-info
Kubernetes master is running at https://localhost:34458
KubeDNS is running at https://localhost:34458/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl get nodes
NAME                       STATUS    ROLES     AGE       VERSION
my-cluster-control-plane   Ready     master    2m        v1.15.3

$ kubectl get po -n kube-system
NAME                                                  READY   STATUS    RESTARTS   AGE
coredns-86c58d9df4-6g66f                              1/1     Running   0          21m
coredns-86c58d9df4-pqcc4                              1/1     Running   0          21m
etcd-my-cluster-control-plane                         1/1     Running   0          20m
kube-apiserver-my-cluster-control-plane               1/1     Running   0          20m
kube-controller-manager-my-cluster-control-plane      1/1     Running   0          20m
kube-proxy-cjgnt                                      1/1     Running   0          21m
kube-scheduler-my-cluster-control-plane               1/1     Running   0          21m
weave-net-ls2v8                                       2/2     Running   1          21m
```

从上面的输出结果，可以看到单节点的 Kubernetes 已经搭建成功。单节点集群默认方式启动的节点类型是 control-plane，该节点包含了所有的组件。这些组件分别是：2*Coredns、Etcd、Api-Server、Controller-Manager、Kube-Proxy、Sheduler 和网络插件 Weave，目前默认使用的网络插件也是 Weave。

### 创建多节点的集群

默认安装的集群只部署了一个控制节点，如果需要部署多节点集群，我们可以通过配置文件的方式来创建多个容器。这样就可以达到模拟多个节点目的，并以这些节点来构建一个多节点的 Kubernetes 集群。

1. 创建多节点 Kubernetes 集群配置文件

Kind 在创建集群的时候，支持通过 --config 参数传递配置文件给 Kind，配置文件可修改的内容主要有 role 和 节点使用的镜像。

- ```
  $ vim my-cluster-multi-node.yaml
  
  # 一共两个节点，一个主节点，一个从节点。
  
  kind: Cluster
  apiVersion: kind.sigs.k8s.io/v1alpha3
  nodes:
  
  - role: control-plane
  - role: worker
  ```

  

2. 创建多节点 Kubernetes 集群

配置文件创建完成后，就可以使用下面的命令来完成多节点 Kubernetes 集群搭建。

```
$ kind create cluster --config my-cluster-multi-node.yaml --name my-cluster-multi-node
Creating cluster "my-cluster-multi-node" ...
 ✓ Ensuring node image (kindest/node:v1.15.3) 

                                                ✓ Preparing nodes  
 ✓ Creating kubeadm config  
 ✓ Starting control-plane ️ 
 ✓ Joining worker nodes  
Cluster creation complete. You can now use se  can now use  use the cluster with:

export KUBECONFIG="$(kin --name="my-cluster-multi-node")"
kubectl cluster-info
```

和上面创建的单节点集群一样，我们同样根据上面命令执行完后，输出的提示信息进行操作来验证一下集群是否部署成功。

```
$ kubectl get nodes
NAME                                  STATUS   ROLES    AGE     VERSION
my-cluster-multi-node-control-plane   Ready    master   3m20s   v1.15.3
my-cluster-multi-node-worker          Ready    <none>   3m8s    v1.15.3

$ kubectl get po -n kube-system
NAME                                                          READY   STATUS    RESTARTS   AGE
coredns-86c58d9df4-cnqhc                                      1/1     Running   0          5m29s
coredns-86c58d9df4-hn9mv                                      1/1     Running   0          5m29s
etcd-my-cluster-multi-node-control-plane                      1/1     Running   0          4m24s
kube-apiserver-my-cluster-multi-node-control-plane            1/1     Running   0          4m17s
kube-controller-manager-my-cluster-multi-node-control-plane   1/1     Running   0          4m21s
kube-proxy-8t4xt                                              1/1     Running   0          5m27s
kube-proxy-skd5v                                              1/1     Running   0          5m29s
kube-scheduler-my-cluster-multi-node-control-plane            1/1     Running   0          4m18s
weave-net-nmfq2                                               2/2     Running   1          5m27s
weave-net-srdfw                                               2/2     Running   0          5m29s
```

### 创建高可用 Kubernetes 集群

Kind 也支持搭建高可用的 Kubernetes 集群，创建方式和多节点集群类似，也是通过配置文件来实现。

1. 创建高可用 Kubernetes 集群配置文件

- ```
  $ vim my-cluster-ha.yaml
  
  # 一共六个节点，三个 control-plane 节点，三个 workers 节点
  
  kind: Cluster
  apiVersion: kind.sigs.k8s.io/v1alpha3
  kubeadmConfigPatches:
  
  - |
    apiVersion: kubeadm.k8s.io/v1beta2
    kind: ClusterConfiguration
    metadata:
      name: config
    networking:
      serviceSubnet: 10.0.0.0/16
    imageRepository: registry.aliyuncs.com/google_containers
    nodeRegistration:
      kubeletExtraArgs:
        pod-infra-container-image: registry.aliyuncs.com/google_containers/pause:3.1
  - |
    apiVersion: kubeadm.k8s.io/v1beta2
    kind: InitConfiguration
    metadata:
      name: config
    networking:
      serviceSubnet: 10.0.0.0/16
    imageRepository: registry.aliyuncs.com/google_containers
    nodes:
  - role: control-plane
  - role: control-plane
  - role: control-plane
  - role: worker
  - role: worker
  - role: worker
  ```

  > 这里，我们通过直接在配置文件里使用国内容器镜像源的方式解决了官方容器镜像源不可用的问题，同时也达到了加速集群创建的目的。

2. 创建高可用 Kubernetes 集群

配置文件创建完成后，就可以使用下面的命令来完成高可用 Kubernetes 集群搭建。

```
$ kind create cluster --name my-cluster-ha --config my-cluster-ha.yaml
Creating cluster "my-cluster-ha" ...
 ✓ Ensuring node image (kindest/node:v1.15.3) 

                                                ✓ Preparing nodes 

                                                                    ✓ Starting the external load balancer ⚖️ 
 ✓ Creating kubeadm config 

                             ✓ Starting control-plane ️ 
 ✓ Joining more control-plane nodes 

                                      ✓ Joining worker nodes 

                                                              Cluster creation complete. You can now use the cluster with:

export KUBECONFIG="$(kind get kubeconfig-path --name="my-cluster-ha")"
kubectl cluster-info
master $ export KUBECONFIG="$(kind get kubeconfig-path --name="my-cluster-ha")"
master $ kubectl cluster-info
Kubernetes master is running at https://localhost:44019
KubeDNS is running at https://localhost:44019/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

同样，我们根据上面命令执行完后，输出的提示信息进行操作来验证一下集群是否部署成功。

```
$ kubectl get nodes
NAME                           STATUS   ROLES    AGE     VERSION
my-cluster-ha-control-plane    Ready    master   3m42s   v1.15.3
my-cluster-ha-control-plane2   Ready    master   3m24s   v1.15.3
my-cluster-ha-control-plane3   Ready    master   2m13s   v1.15.3
my-cluster-ha-worker           Ready    <none>   96s     v1.15.3
my-cluster-ha-worker2          Ready    <none>   98s     v1.15.3
my-cluster-ha-worker3          Ready    <none>   95s     v1.15.3

```

从上面的输出结果，可以看到包含了多个 master 节点，说明高可用的 Kubernetes 集群已经搭建成功。

其它相关知识

------

### Kind 的镜像里的秘密

Kind 镜像一共分为两类，一类是 Base 镜像，另一类是 Node 镜像。

1. Base 镜像

Base 镜像目前使用了 ubuntu:19.04 作为基础镜像，并做了下面的调整：

- 安装 Systemd 相关的包，并调整一些配置以适应在容器内运行。
- 安装 Kubernetes 运行时的依赖包，比如: Conntrack、Socat、CNI 等。
- 安装容器运行环境，比如: Containerd、Crictl 等。
- 配置自己的 ENTRYPOINT 脚本，以适应和调整容器内运行的问题。

更多具体的构建逻辑可以参考：https://github.com/kubernetes-sigs/kind/blob/master/images/base/Dockerfile

2. Node 镜像

Node 镜像的构建比较复杂，目前是通过运行 Base 镜像并在 Base 镜像内执行操作，再保存此容器内容为镜像的方式来构建的，包含的操作有：

- 构建 Kubernetes 相关资源，比如：二进制文件和镜像。
- 运行一个用于构建的容器
- 把构建的 Kubernetes 相关资源复制到容器里
- 调整部分组件配置参数，以支持在容器内运行
- 预先拉去运行环境需要的镜像
- 通过 docker commit 方式保存当前的构建容器为 Node 镜像

### 如何快速删除一个集群

如果你不需要本地的集群环境，通过以下命令进行删除：

```
kind delete  cluster --name my-cluster
Deleting cluster "my-cluster" ...
$KUBECONFIG is still set to use /root/.kube/kind-config-my-cluster even though that file has been deleted, remember to unset it

```

至此，我们就演示完了如何使用 Kind 快速搭建一个 Kubernetes 集群。不过有一个你需要注意的地方，Kind 搭建的集群不适用于生产环境中使用。但是如果你想在本地快速构建一个 Kubernetes 集群环境，并且不想占用太多的硬件资源，那么 Kind 会是你不错的选择