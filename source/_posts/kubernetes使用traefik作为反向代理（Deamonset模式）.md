cover: http://ciwei2.cn-sh2.ufileos.com/71.jpg
title: kubernetes使用traefik作为反向代理（Deamonset模式）
date: 2019-06-12 16:26:00
tags: [kubernetes,traefik]
categories: [综合]
---
### 介绍

 traefik  在Kubernetes中的部署方式有2种: Deployment 以及DaemonSet。区别主要是：

相比一个节点只部署一个daemonset的traefik，采用deployment会更易于伸缩和扩展；

Daemonset可以利用taints和tolerations字段在自定义的节点上部署traefik服务；

采用Daemonset方式，可以在任何节点上访问80和443端口，而使用deployment者必须依赖service里面定义的对象去访问，端口使用Nodeport模式暴露

使用 traefik 后所有接口都可以通过80访问 因为代理了

<!--more-->

### 使用DaemonSet模式部署traefik(在kubernetes master节点)

kubernetes环境

|  主机名称   | ip地址	  |  操作系统	   | 角色  |  备注   |
|  ----  | ----  |  ----  | ----  |  ----  |
| master  | 10.99.12.203 |CentOS 7.5  | proxy, master，traefik |DaemonSet  |
| node1  | 10.99.12.204 | CentOS 7.5  | worker | 无 |
| node2  | 10.99.12.205 | CentOS 7.5  | worker | 无 |

### 从GitHub下载treafik部署文件

```java
mkdir -p kubernetes/traefik
cd kubernetes/traefik
wget https://github.com/containous/traefik/archive/v1.7.12.zip
unzip v1.7.12.zip
cd traefik-1.7.12/examples/k8s/
# tree
	.
	├── cheese-default-ingress.yaml
	├── cheese-deployments.yaml
	├── cheese-ingress.yaml
	├── cheese-services.yaml
	├── cheeses-ingress.yaml
	├── traefik-deployment.yaml
	├── traefik-ds.yaml          # DaemonSet模式文件
	├── traefik-rbac.yaml        # rbac
	└── ui.yaml                  # 使用treafik代理traefik-ui的配置文件
```

这里使用的DaemonSet，只是用traefik-ds.yaml ，traefik-rbac.yaml ， ui.yaml

### 部署traefik

rabc

```java
kubectl apply -f traefik-rbac.yaml

kubectl get clusterrole
```

treafik

github上下载的部署文件traefik-ds.yaml,会在所有节点部署traefik，而我们只需要在1个master节点部署，因此需要添加nodeSelector过滤，如果不需要只部署master这一步可以省略呀

```java
cat traefik-ds.yaml

  #...

kind: DaemonSet
apiVersion: extensions/v1beta1
# ...
spec:
  template:
    # ...
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      #=================添加nodeSelector信息：只在master节点创建===================
      tolerations:
      - key: node-role.kubernetes.io/master
        operator: "Equal"
        value: ""
        effect: NoSchedule
      nodeSelector:
        node-role.kubernetes.io/master: ""
      #=========================================================================
      containers:
      - image: traefik
        name: traefik-ingress-lb
      #...

```

部署

```java
kubectl apply -f traefik-ds.yaml

kubectl get svc -n kube-system

kubectl get pods -n kube-system -o wide
```

访问测试

以上treafik daemonset方式部署完成，可以通过master_ip:8080访问traefik-ui，例如10.99.12.201:8080
可以看到目前里面什么都没有，因为我们只是配置好了traefik，但是还没有使用

![](/images/20190211160709311.png)

### traefik-ui使用traefik进行代理（相当于一个demo实验）

上面使用的是master_ip:8080(traefik_ui服务端口，可以修改)，现在我们使用traefik来代理traefik-ui，我们使用域名traefik-ui.ejuops.com(该域名不存在，使用配置hosts的方式访问)

### 配置traefik代理traefik-ui

```java
# cat ui.yaml
# 具体内容根据实际情况修改，例如port，name，host等。
---
apiVersion: v1
kind: Service
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
  - name: web
    port: 80
    targetPort: 8080
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  rules:
  # host根据实际情况进行修改。
  - host: traefik-ui.ejuops.com
    http:
      paths:
      - path: /
        backend:
          serviceName: traefik-web-ui
          servicePort: web
```

部署

```java
kubectl apply -f ui.yaml

kubectl get svc -n kube-system

kubectl get ingress -n kube-system
```

### 配置test-walle的项目

配合idea的docker发布到服务器

镜像标签为：demo:1.0

https://github.com/ciweigg2/test-walle.git

vi deploy-demon.yaml

```java
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: default
spec:
  selector:
    app: myapp
    release: canary
  ports:
  - name: http
    port: 8080
    targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      release: canary
  template:
    metadata:
      labels:
        app: myapp
        release: canary
    spec:
      containers:
      - name: myapp
        image: demo:1.0
        ports:
        - name: httpd
          containerPort: 8080
```

创建myapp的traefik规则

因为demo:1.0的镜像默认是根路径 所以设置/ 否则都是404 如果springboot设置了路径那这边也可以需要设置对应路径

- host: myapp.ciwei.com 如果配置成 - host: 说明可以使用ip地址和任何指向这个服务器的域名访问

vi traefik-myapp.yaml 

```java
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-myapp
  namespace: default
  annotations:
    ingress.kubernetes.io/ssl-redirect: "false" ##关闭强制使用HTTPS的设置
spec:
  rules:
  ## 根据域名实现转发
  - host: myapp.ciwei.com
    http:
      paths:
      - path: /
        backend:
          serviceName: myapp
          servicePort: 8080
```

部署并且查看状态

```java
kubectl apply -f deploy-demon.yaml
kubectl apply -f traefik-myapp.yaml
kubectl get pods
```

### 在主机上配置hosts文件（浏览器主机）

C:\Windows\System32\drivers\etc\hosts

```java
10.99.12.201 traefik-ui.ejuops.com
10.99.12.201 myapp.ciwei.com
```

### 通过域名访问traefik-ui

http://traefik-ui.ejuops.com 可以看到treafik-ui的代理情况（frontends,backends,Health），目前只有这个使用使用了代理，因为我还部署了个test-walle的项目 所以看得到呀

访问http://traefik-ui.ejuops.com可以访问控制台

访问http://myapp.ciwei.com/walle可以访问接口

![](/images/20190612171144.png)