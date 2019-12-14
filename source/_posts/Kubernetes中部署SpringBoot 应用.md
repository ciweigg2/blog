cover: http://ciwei2.cn-sh2.ufileos.com/70.jpg
title: Kubernetes中部署SpringBoot 应用
date: 2019-06-01 13:13:06
tags: [kubernetes]
categories: [综合]
---
### 创建Dockerfile(把自己的项目部署到服务器)

如果多个replicas的话 就是负载均衡了

```java
FROM java:8
VOLUME /tmp
ADD /target/test-walle-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java","-jar","-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005","/app.jar"]
```

<!--more-->

### 在 Kubernetes 中添加服务

kubernetes会自动检测服务是否正常参考[Liveness](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)

方式1 http需要保证path路径能够访问

```java
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

方式2 tcp需要保证端口可以通

```java
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

vi k8s-demo.yaml

修改镜像名：image: registry.cn-qingdao.aliyuncs.com/hellowoodes/k8s-service

```java
apiVersion: v1
kind: Service
metadata:
  name: k8s-service
  namespace: default
  labels:
    app: k8s-service
spec:
  type: NodePort
  ports:
  - port: 8080
    nodePort: 30002
  selector:
    app: k8s-service

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-service
  labels:
    app: k8s-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: k8s-service
  template:
    metadata:
      labels:
        app: k8s-service
    spec:
      containers:
      - name: k8s-service
        image: registry.cn-qingdao.aliyuncs.com/hellowoodes/k8s-service
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
        livenessProbe:
          tcpSocket:
            port: 8080
          periodSeconds: 15
          initialDelaySeconds: 30
```

### 创建服务

kubectl apply -f k8s-demo.yaml 

等待服务启动之后访问 ${NodeIP}:30002/k8s，会返回 Hello Kubernetes，部署完成

也可以直接在kubernetes界面中右侧点击创建 输出yaml的内容