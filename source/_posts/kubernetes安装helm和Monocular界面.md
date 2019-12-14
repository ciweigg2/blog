cover: http://ciwei2.cn-sh2.ufileos.com/73.jpg
title: kubernetes安装helm和Monocular界面
date: 2019-06-05 18:14:55
tags: [kubernetes,helm,monocular]
categories: [综合]
---
### 安装helm

<!--more-->

前提需要安装了kubernetes

Monocular界面可以管理helm安装的所有Charts，很方便的呀

```java

安装helm

wget https://storage.googleapis.com/kubernetes-helm/helm-v2.14.0-linux-amd64.tar.gz
tar zxvf helm-v2.14.0-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/
helm version

安装 tiller server

为tiller绑定管理角色cluster-admin,使tiller拥有集群集群级别最高权限：

vi tiller-rbac.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system

kubectl create -f tiller-rbac.yaml

对应helm版本

helm init --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.14.0 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```

部署完成之后使用 “kubectl get pods” 命令确认tiller Pod 运行正常；

```java
kubectl get pods -n kube-system -l app=helm
```

查看版本：

```java
helm version
Client: &version.Version{SemVer:"v2.14.0", GitCommit:"618447cbf203d147601b4b9bd7f8c37a5d39fbb4", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.0", GitCommit:"7cf31e8d9a026287041bae077b09165be247ae66", GitTreeState:"clean"}
```

更换仓库源

```java
helm repo remove stable
helm search
helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
helm repo update
```

安装charts

Monocular是一个开源软件，用于管理kubernetes上以Helm Charts形式创建的服务

### helm授权

```java
kubectl create clusterrolebinding add-on-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:default
```

### 安装charts web ui (可能会很慢 需要等待的)

参考：https://www.cnblogs.com/kuku0223/p/9173953.html

添加chart仓库

```java
helm repo add cherryleo https://fileserver-1253732882.cos.ap-chongqing.myqcloud.com/charts
```

安装ingress

```java
helm install cherryleo/nginx-ingress --set controller.image.repository=ccr.ccs.tencentyun.com/quay.io/nginx-ingress-controller --set controller.image.tag=0.15.0 --set defaultBackend.image.repository=ccr.ccs.tencentyun.com/k8s.io/defaultbackend --set defaultBackend.image.tag=1.4
```

安装monocular

```java
helm install cherryleo/monocular --name monocular
```

查看monocular(需要等待的)

```java
kubectl get pods
NAME                                                              READY     STATUS    RESTARTS   AGE
giggly-grasshopper-nginx-ingress-controller-8965d964f-58zwr       1/1       Running   0          2h
giggly-grasshopper-nginx-ingress-default-backend-58954bc7chdvxz   1/1       Running   0          2h
monocular-mongodb-6c9b897fdf-nh49n                                1/1       Running   0          4m
monocular-monocular-api-5697776746-h4j9z                          1/1       Running   2          4m
monocular-monocular-prerender-6866b8c48d-8zdqt                    1/1       Running   0          4m
monocular-monocular-ui-75f899bf44-x96hr                           1/1       Running   0          4m
```

查看ingress端口

monocular必须通过ingress进行访问

```java
kubectl get svc | grep ingress
romping-indri-nginx-ingress-controller        LoadBalancer   10.108.156.209   <pending>     80:30925/TCP,443:32022/TCP   3h45m
romping-indri-nginx-ingress-default-backend   ClusterIP      10.99.157.120    <none>        80/TCP                       3h45m
```

访问：https://ip:32022

添加仓库：

stable	
https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts	
https://github.com/kubernetes/charts/tree/master/stable

lasdadeasds    
https://kubernetes-charts-incubator.storage.googleapis.com/    
https://github.com/kubernetes/charts/tree/master/stable

![](/images/20190605214433.png)

需要等待一段时间加载仓库 然后刷新就好

![](/images/20190605214229.png)