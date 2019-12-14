cover: http://ciwei2.cn-sh2.ufileos.com/138.jpg
title: 使用Kubespray部署k8s集群
date: 2018-09-15 22:48:41
tags: [kubernetes,k8s]
categories: [综合]
---
### 一、准备工作

1.规划集群

> 集群节点规划 

> 10.254.4.1

> 10.254.4.2

> 10.254.4.3

<!--more-->

2. 关闭swap

```
swapoff -a
```

3. 节点互信

本文以第一个节点 10.254.4.1作为操作节点，所以需要做从10.254.4.1 到其他节点的互信，在4.1上执行如下命令：

> ssh-keygen -t rsa

> ssh-copy-id -i /root/.ssh/id_rsa.pub root@10.254.4.1

> ssh-copy-id -i /root/.ssh/id_rsa.pub root@10.254.4.2

> ssh-copy-id -i /root/.ssh/id_rsa.pub root@10.254.4.3

4.软件安装

安装以下可能会用到的软件：

```
//可能python使用yum会装不到  
yum install -y epel-release python-pip python34 python34-pip ansible    
pip install netaddr    
pip install --upgrade pip    
// 这个组件版本不对，会导致kubespray安装失败，需要更新  
pip install --upgrade jinja2
```

### 二、使用kubespray部署集群

```
1. 下载kubespray

wget https://github.com/kubernetes-incubator/kubespray/archive/v2.6.0.tar.gz

2. 节点规划配置

tar -xvf v2.6.0.tar.gz
cd kubespray-2.6.0/   
cp -rfp inventory/sample inventory/mycluster   
declare -a IPS=(10.254.4.1 10.254.4.2 10.254.4.3)   
CONFIG_FILE=inventory/mycluster/hosts.ini python3 contrib/inventory_builder/inventory.py ${IPS[@]}

3. 修改国内源

 安装过程中会用到google的镜像，此处替换为使用国内源，执行：

cd kubespray/   
sed -i  's/gcr\.io\/google_containers/mirrorgooglecontainers/g' roles/download/defaults/main.yml  
sed -i 's/gcr\.io\/google-containers/mirrorgooglecontainers/g' roles/download/defaults/main.yml  
sed -i 's/gcr\.io\/google_containers/mirrorgooglecontainers/g' roles/kubernetes-apps/ansible/defaults/main.yml

4. 安装
ansible-playbook -i inventory/mycluster/hosts.ini cluster.yml

5. 安装成功

=============================================================================== 
download : contain--------------------------------- 190.31s
network_plugin/calico : Calico | Copy cni plugins from hyperkube -------------------------------------------------- 182.77s
download : container_download | Download containers if pull is required or told to always pull (all nodes) ------------------------------------------------------------------------ 141.82s
download : container_download ---------------------- 92.60s
kubernetes/preinstall : Update package management cache (YUM) ------------------------------------------------------ 36.63s
network_plugin/calico : Calico | Copy cni plugins from calico/cni container ---------------------------------------- 33.25s
download : container_download | Download ccntainers if pull is required or told to always pull (all nodes)---------- 30.09s
download : container_download | Download containers if pull is required or told to always pull (all nodes) --------- 27.44s
gather facts from all instances -------------------- 24.79s
kubernetes/master : Master | wait for the apiserver to be running -------------------------------------------------------------------------------------------------                  22.32s
kubernetes/preinstall : Install packages requirements -------------------------------------------------------------- 22.266
download : container_download | Download containers if pull is required or told to always pull (all nodes) ----------------------------------------------------------------          20.79s
ettd : Configure | Join member(s) to etcd cluster one at a time ---------------------------------------------------- 20.11s
etcd : Configure | Join member(s) to etcd-events cluster one at a timee------------------------------------------------------------------------------------------------------------  20.11s
download : container_download | Download containers if pull is required or told to always pull (all nodes) ------------------------------------------------------------------------ 19.888s
download : connainer_download | Download containers if pull is required or told to always pull (all nodes) ------------------------------------------------------------------------- 12.43s
download : container_download | Download containers if pull is required or told to always pull (all nodes) ------------------------------------------------------------------------- 11.20s
etcd : reload etcd --------------------------------- 10.76s
docker : Docker | pause while Docker restarts -------------------------------------------------------------------------------------------------------------------------------------- 10.17s
download : container_download | Download containers if pull is required or told to always pull (all nodes) -------------------------------------------------------------------------- 9.08s

6. 验证

[root@test kubespray-2.5.0]# kubectl get nodes
NAME      STATUS     ROLES         AGE       VERSION
node1     Ready      master,node   9m        v1.9.5
node2     Ready      master,node   9m        v1.9.5
node3     Ready      node          9m        v1.9.5
[root@test kubespray-2.5.0]# kubectl get pods --all-namespaces
NAMESPACE     NAME                                    READY     STATUS     RESTARTS   AGE
kube-system   calico-node-6qvg2                       1/1       Running    0          5m
kube-system   calico-node-hjw47                       1/1       Running    0          5m
kube-system   calico-node-kcwxf                       1/1       Running    0          5m
kube-system   kube-apiserver-node1                    1/1       Running    0          9m
kube-system   kube-apiserver-node2                    1/1       Running    0          9m
kube-system   kube-controller-manager-node1           1/1       Running    0          10m
kube-system   kube-controller-manager-node2           1/1       Running    0          10m
kube-system   kube-dns-5dbd8b996b-6g89g               3/3       Running    0          5m
kube-system   kube-dns-5dbd8b996b-nd98t               3/3       Running    0          5m
kube-system   kube-proxy-node1                        1/1       Running    0          9m
kube-system   kube-proxy-node2                        1/1       Running    0          9m
kube-system   kube-scheduler-node1                    1/1       Running    0          10m
kube-system   kube-scheduler-node2                    1/1       Running    0          10m
kube-system   kubedns-autoscaler-d76d58748-gql8t      1/1       Running    0          5m
kube-system   kubernetes-dashboard-6c96b87867-tk557   1/1       Running    0          5m
```

> 至此，集群安装成功，由于kubespray安装默认自带了dashboard，可以直接使用了

验证：部署一个NGINX

```
# 启动一个单节点nginx
kubectl run nginx --image =nginx:1.7.9 --port=80
# 为“nginx”服务暴露端口
kubectl expose deployment nginx  -- type = NodePort
# 查看nginx服务详情
kubectl  get  svc nginx
NAME      TYPE       CLUSTER - IP     EXTERNAL-IP   PORT ( S )         AGE 
nginx      NodePort  10.233 . 29.96   <none>        80 : 32345 / TCP   15s
# 访问测试，如果能够正常返回NGINX首页，说明正常
curl localhost : 32345
```

卸载:

```
ansible - playbook  - i inventory / mycluster / hosts . ini reset . yml
```
