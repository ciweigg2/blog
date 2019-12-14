cover: http://ciwei2.cn-sh2.ufileos.com/128.jpg
title: traefik金丝雀部署
date: 2019-06-13 17:12:32
tags: [traefik]
categories: [综合]
---
### 介绍

可以使用服务权重在多个部署之间以细粒度方式拆分Ingress流量。

一个规范用例是canary版本，其中代表较新版本的部署将随着时间的推移接收最初较小但不断增加的请求部分。在Traefik中可以这样做的方法是指定应该进入每个部署的请求的百分比。

<!--more-->

例如，假设一个应用程序my-app在版本1中运行。较新的版本2即将发布，但对生产中运行的新版本的稳健性和可靠性的信心只能逐渐获得。因此，my-app-canary创建新部署并将其缩放到足以获得1％流量份额的副本计数。与此同时，像往常一样创建一个Service对象

### 测试首先需要准备2个服务 一个是my-app 一个是my-app-canary

Ingress规范看起来像这样：

vi traefik-myapp.yaml

```java
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    traefik.ingress.kubernetes.io/service-weights: |
      my-app: 99%
      my-app-canary: 1%
  name: traefik-myapp
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: my-app
          servicePort: 8080
        path: /
      - backend:
          serviceName: my-app-canary
          servicePort: 8080
        path: /
```

记下traefik.ingress.kubernetes.io/service-weights注释：它指定引用的后端服务之间的请求分配，my-app以及my-app-canary。根据这一定义，Traefik将99％的请求路由到my-app部署支持的pod ，并将1％的请求路由到支持的pod my-app-canary。随着时间的推移，该比例可能会慢慢转向金丝雀部署，直到它被认为取代之前的主要应用程序，步骤如5％/ 95％，10％/ 90％，50％/ 50％，最后100％/ 0％。

必须满足一些条件才能正确应用服务权重：

> 关联的服务后端必须共享相同的路径和主机 path: / 相同

> 所有服务后端共享的总百分比必须达到100％（但请参阅省略最终服务的部分）

> 百分比值被解释为浮点数到支持的精度，如[注释文档中](https://docs.traefik.io/configuration/backends/kubernetes#general-annotations)中所定义

省略最终服务

指定服务权重时，出于方便原因，可以省略一个服务。

例如，以下定义显示了如何在金丝雀发布伴随基线部署的情况下拆分请求，以便更轻松地进行指标比较或自动化金丝雀分析：

```java
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    traefik.ingress.kubernetes.io/service-weights: |
      my-app-canary: 10%
      my-app-baseline: 10%
  name: app
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: my-app-canary
          servicePort: 8080
        path: /
      - backend:
          serviceName: my-app-baseline
          servicePort: 8080
        path: /
      - backend:
          serviceName: my-app-main
          servicePort: 8080
        path: /
```
        
此配置my-app-main自动分配80％的流量，从而使用户无需手动完成百分比值。当连续增加金丝雀释放的份额时，这变得很方便

### 踩坑：

traefik.ingress.kubernetes.io/affinity: "true" 这个不能用啊 用的话cookie会保持的 所以流量不会引入新的规则

如果配置了必须每次把之前cookie清除了 否则就算改了流量也会发送到之前请求的地址 不建议这样呀

```java
apiVersion: v1
kind: Service
metadata:
  name: myapp2
  namespace: default
  annotations:
   # traefik.ingress.kubernetes.io/affinity: "true"
```

参考：https://docs.traefik.io/user-guide/kubernetes/#omitting-the-final-service中的Traffic Splitting