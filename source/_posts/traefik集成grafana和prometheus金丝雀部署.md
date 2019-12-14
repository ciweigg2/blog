cover: http://ciwei2.cn-sh2.ufileos.com/129.jpg
title: traefik集成grafana和prometheus金丝雀部署
date: 2019-06-15 22:59:17
tags: [traefik,prometheus,grafana,kubernetes]
categories: [综合]
---
### 介绍

需要提前准备好kubernetes

部署好grafana prometheus

部署好traefik

<!--more-->

### 创建2个版本的服务

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
    release: 1.0.1
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
      release: 1.0.1
  template:
    metadata:
      labels:
        app: myapp
        release: 1.0.1
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/path: /actuator/prometheus
        prometheus.io/port: "8080"
    spec:
      containers:
      - name: myapp
        image: demo:1.0
        ports:
        - name: httpd
          containerPort: 8080
```

vi deploy-demon2.yaml 

```java
apiVersion: v1
kind: Service
metadata:
  name: myapp2
  namespace: default
spec:
  selector:
    app: myapp2
    release: 1.0.2
  ports:
  - name: http
    port: 8080
    targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy2
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp2
      release: 1.0.2
  template:
    metadata:
      labels:
        app: myapp2
        release: 1.0.2
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/path: /actuator/prometheus
        prometheus.io/port: "8080"
    spec:
      containers:
      - name: myapp2
        image: demo:1.0
        ports:
        - name: httpd
          containerPort: 8080
```

服务myapp 1.0.1和myapp2 1.0.2

kubectl create -f deploy-demon.yaml

kubectl create -f deploy-demon2.yaml

### 部署traefik的Ingress

vi traefik-myapp.yaml 

```java
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    traefik.ingress.kubernetes.io/service-weights: |
      myapp: 80%
      myapp2: 20%
  name: traefik-myapp
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: myapp
          servicePort: 8080
        path: /
      - backend:
          serviceName: myapp2
          servicePort: 8080
        path: /
```

创建规则呀

```java
kubectl apply -f traefik-myapp.yaml
```

访问：http://ip/walle

grafana监控查看流量

现在配置的是myapp 80% myapp2 20% 可以看出流量基本控制在myapp上的呀

![](/images/20190615230815.png)

修改流量配置 可以在kubernetes界面上修改但是感觉不太好 配置文件修改比较好

更新Ingress规则(不需要delete然后create，apply直接动态更新配置的呀)

我把流量改成myapp 0% myapp2 100%

```java
kubectl apply -f traefik-myapp.yaml
```

输出ingress.extensions/traefik-myapp configured表示更新成功呀

![](/images/20190615231402.png)

这样流量全部转到新的版本上了 可以删除旧的版本了

kubectl delete -f deploy-demon.yaml

金丝雀部署完成了呀，是不是很简单