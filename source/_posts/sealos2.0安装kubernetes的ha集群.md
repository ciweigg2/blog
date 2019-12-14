title: sealos2.0安装kubernetes的ha集群
date: 2019-08-01 21:22:50
tags: [sealos,kubernetes]
categories: [综合]
---
# Sealos 2.0

支持kubernetes 1.14.0 以上版本，HA不再依赖keeplived与haproxy, 通过ipvs直接代理masters节点

通过lvscare健康检测masters, 是一种非常先进且稳定的HA方式。安装失败率极低

<!--more-->

参考：https://github.com/fanux/sealos

# 快速使用

## 准备条件

* 装好docker并启动docker
* 把[离线安装包](https://github.com/sealstore/cloud-kernel/releases/) 下载好拷贝到执行节点的任意目录,不需要解压,sealos会自动检测各个节点是否有安装包,若不存在则会scp到该节点。如果有文件服务器更好，sealos也支持从一个服务器上wget到所有节点上。 离线包中sealos暂不支持scp，请到release界面下载最新版sealos

[sealos](https://github.com/fanux/sealos/releases)其实sealos文件在离线安装包的bin目录下面有可以不下载

离线包v1.15.0已经上传到网盘 kubeadm是sealos定制的和其他的不一样的所以称为超级kubeadm

## 安装docker

安装依赖包

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

设置stable repository

```
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

查看可用版本的docker-ce

```
yum list docker-ce --showduplicates | sort -r
```

安装

```
yum install docker-ce-18.09.0
```

启动docker

```
设置开机启动
systemctl enable docker.service
启动服务呀
systemctl start docker
```

## 安装

sealos已经放在离线包中，解压后在kube/bin目录下(可以解压一个，获取sealos bin文件)

```
sealos init \
    --master 192.168.0.2 \
    --master 192.168.0.3 \
    --master 192.168.0.4 \                    # master地址列表
    --node 192.168.0.5 \                      # node地址列表
    --user root \                             # 服务用户名
    --passwd your-server-password \           # 服务器密码，用于远程执行命令
    --pkg-url /root/kube1.15.0.tar.gz  \      # 离线安装包位置，可支持http/https服务器（http://store.lameleg.com/kube1.15.0.tar.gz）存放和本地（/root/kube1.15.0.tar.gz）存放两种方式。若对应节点上文件不存在则会从执行机器上scp文件到对应节点。
    --version v1.15.0                        # kubernetes 离线安装包版本，这渲染kubeadm配置时需要使用
```

上面输出的--experimental-control-plane --certificate-key 33b0ea9ea245ac5cb6ee69be3c939f15f443b18656b0c01d78b3720d2e3ceeb9一定要记住 不能再次获取 这个是加入新master必要的参数

其它参数:

```
 --kubeadm-config string   kubeadm-config.yaml local # 自定义kubeadm配置文件，如有这个sealos就不去渲染kubeadm配置
 --vip string              virtual ip (default "10.103.97.2") # 代理master的虚拟IP，只要与你地址不冲突请不要改
```

## 使用sealos的init命令初始化的

假设在192.168.0.2机器上执行sealos init命令的 那么所有其他master机器都需要执行

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

这样执行后其他master机器才能使用kubectl get node命令 集群必须3台或者以上的机器 否则停止了一台后会导致整个集群有问题 三台机器中停止一台集群还可以正常工作 但是再停止一台只有一台正常的情况下整个集群无法工作 至少有2台机器是正常的才能工作呀

## 安装完成后查看集群状态

```
kubectl get node
NAME                      STATUS   ROLES    AGE     VERSION
izj6cdqfqw4o4o9tc0q44rz   Ready    master   2m25s   v1.15.0
izj6cdqfqw4o4o9tc0q44sz   Ready    master   119s    v1.15.0
izj6cdqfqw4o4o9tc0q44tz   Ready    master   63s     v1.15.0
izj6cdqfqw4o4o9tc0q44uz   Ready    <none>   38s     v1.15.0
```

## 清理
```
sealos clean \
    --master 192.168.0.2 \
    --master 192.168.0.3 \
    --master 192.168.0.4 \          # master地址列表
    --node 192.168.0.5 \            # node地址列表
    --user root \                   # 服务用户名
    --passwd your-server-password
```

## 增加master节点

假设新master节点ip是：192.168.0.8

master节点加入参考：https://sealyun.com/post/super-kubeadm/

需要将离线包上传到新节点 然后解压呀

```
进入解压后的目录
cd kube/shell && init.sh
将其中一个master的节点写入hosts文件中呀 因为kubelet kube-proxy配置的都是这个解析名,如果不改解析master宕机整个集群就不可用了
echo "192.168.0.2 apiserver.cluster.local" >> /etc/hosts
查看token
kubeadm token create --print-join-command
加入新的master节点 下面--master参数为什么是三个呢 因为上面初始化了3个master节点 如果一个就填写一个
kubeadm join 192.168.0.2:6443 --token 9vr73a.a8uxyaju799qwdjv \
    --master 192.168.0.2 \
    --master 192.168.0.3 \
    --master 192.168.0.4 \ 
    --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866 --experimental-control-plane --certificate-key 33b0ea9ea245ac5cb6ee69be3c939f15f443b18656b0c01d78b3720d2e3ceeb9
解析改成自己ip
sed "s/192.168.0.2/192.168.0.8/g" -i /etc/hosts
```

### 配置管理呀

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 在任意master节点查看集群状态

```
kubectl get node
```

## 增加node节点

需要将离线包上传到新节点 然后解压呀

注意 初始化的时候只初始化了3个master节点 如果后面添加了master节点 那么--master也需要添加新的master节点

```
进入解压后的目录
cd kube/shell && init.sh
echo "10.103.97.2 apiserver.cluster.local" >> /etc/hosts   # using vip
kubeadm join 10.103.97.2:6443 --token 9vr73a.a8uxyaju799qwdjv
    --master 192.168.0.2 \
    --master 192.168.0.3 \
    --master 192.168.0.4 \ 
    --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866
```

## 删除master或者node节点

```
kubectl delete node jd2
```

如果删除的是node节点 那么node节点需要systemctl stop kubelet

如果删除的是master节点 那么master节点需要kubeadm reset

## 安装dashboard prometheus等
离线包里包含了yaml配置和镜像，用户按需安装。
```
cd /root/kube/conf
kubectl taint nodes --all node-role.kubernetes.io/master-  # 去污点，根据需求看情况，去了后master允许调度
kubectl apply -f heapster/ # 安装heapster, 不安装dashboard上没监控数据
kubectl apply -f heapster/rbac 
kubectl apply -f dashboard  # 装dashboard
kubectl apply -f prometheus # 装监控
```

如果集群安装有问题可能是LB出问题了

安装ipvsadm

```
yum install ipvsadm
```

查看lb规则

```
ipvsadm -Ln
```

清理规则呀(每个节点呀)

```
ipvsadm -C
```

# 原理
```
  +----------+                       +---------------+  virturl server: 127.0.0.1:6443
  | mater0   |<----------------------| ipvs nodes    |    real servers:
  +----------+                      |+---------------+            10.103.97.200:6443
                                    |                             10.103.97.201:6443
  +----------+                      |                             10.103.97.202:6443
  | mater1   |<---------------------+
  +----------+                      |
                                    |
  +----------+                      |
  | mater2   |<---------------------+
  +----------+
```
sealos 只是帮助用户去渲染配置远程执行命令，低层依赖两个东西，一个是lvscare，一个是定制化的超级kubeadm

lvscare原理参考：https://github.com/fanux/LVScare