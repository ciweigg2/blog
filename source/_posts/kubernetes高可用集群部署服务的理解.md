title: kubernetes高可用集群部署服务的理解
date: 2019-08-03 22:10:12
tags: [kubernetes]
categories: [综合]
---
### 理解k8s集群

sealos部署kubernetes高可用集群

必须3台或者以上的master才能搭建高可用集群 集群数目小于2的话整个集群会宕机的

<!--more-->

概念：宕机的机器会随着机器的恢复自动加入集群

### k8s使用高可用部署服务

简单的例子

名称 | 地址 |  角色  
-|-|-
master01 | 192.168.0.1 | master
master02 | 192.168.0.2 | master
master03 | 192.168.0.3 | master
node01 | 192.168.0.4 | node
node02 | 192.168.0.5 | node
node03 | 192.168.0.6 | node

在master01上测试部署nginx服务，nginx服务成功部署到node2上

```
$ kubectl run nginx --image=nginx --port=80
deployment "nginx" created

$ kubectl get pod -o wide -l=run=nginx
NAME                     READY     STATUS    RESTARTS   AGE       IP           NODE
nginx-2662403697-pbmwt   1/1       Running   0          5m        10.244.7.6   node3
```

在master01让nginx服务外部可见

```
$ kubectl expose deployment nginx --port=80 --target-port=80 --type=NodePort
service "nginx" exposed

$ kubectl get svc -l=run=nginx
NAME      CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx     10.105.151.69   <nodes>       80:31639/TCP   43s
```

访问三个master其中一个都会显示nginx的首页 把这三个nginx内网ip使用云服务的负载均衡 或者nginx将3个master内网ip集中起来使用外网ip暴露 达到高可用外网访问了

192.168.0.1:31639

192.168.0.2:31639 ->  nginx代理 外网ip访问(或者使用云服务提供的负载均衡呀)

192.168.0.3:31639

```
$ curl master02:31639
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### 总结

sealos很好的展示了集群master的概念

参考高可用集群的测试：https://gitee.com/cookeem/kubeadm-ha

正确的架构应该是这样的吧 master内网集群 node节点暴露使用LB负载均衡

![](/images/image-20190713075717350.df5244cd.png)