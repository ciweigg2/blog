cover: http://ciwei2.cn-sh2.ufileos.com/75.jpg
title: kubernetes常用的命令
date: 2019-06-13 18:08:53
tags: [kubernetes]
categories: [综合]
---
### kubectl命令

kubernetes常用的命令

<!--more-->

得到所有命名空间

```java
kubectl get namespaces
```

同理得到所有的节点

```java
kubectl get nodes
```

删除weave命名空间

```java
kubectl delete namespace weave
```

得到monitoring命名空间中所有的pod，不加-n monitoring得到的是default命名空间下的数据。

```java
kubectl get pods -n monitoring
```

同理Deployment

```java
kubectl get deployments -n monitoring
```

得到Pod的详细信息，包括其所属节点的信息

```java
kubectl get pods -o wide -n monitoring
```

得到monitoring命名空间下某pod的详细信息

***为“3”得到的某个pod的名称

```java
kubectl describe pods *** -n monitoring
```

查看monitoring命名空间下某pod的日志

***为“3”得到的某个pod的名称

```java
kubectl logs *** -n monitoring
```

使用yaml文件为weave 命名空间创建一个资源

```java
kubectl create --namespace weave -f scope.yaml 
```

查看monitoring命名空间下所有service的详细信息（包含端口信息）

```java
kubectl get svc -n monitoring 
```

删除k8s集群中的node3节点

```java
kubectl delete node node3
```

kubectl apply -f 使对yaml配置文件的修改生效