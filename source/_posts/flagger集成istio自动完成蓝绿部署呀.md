cover: http://ciwei2.cn-sh2.ufileos.com/37.jpg
title: flagger集成istio自动完成蓝绿部署
date: 2019-07-01 21:45:02
tags: [flagger,istio]
categories: [综合]
---
### 介绍

接着上一篇文章：flagger集成istio自动完成金丝雀部署

这边文章主要讲蓝绿部署

所有的部署最好是在app新版之前发布验证是否会影响老用户部署的，千万别再app新版发布后发布接口，这样可能会导致接口404的情况的所以要注意的哟

<!--more-->

### 部署

服务啊网关啊都用上一篇文章的

vi bluegreen.yaml

```
apiVersion: flagger.app/v1alpha3
kind: Canary
metadata:
  name: podinfo
  namespace: test
spec:
  # service mesh provider can be: kubernetes, istio, appmesh, nginx, gloo
  # use the kubernetes provider for Blue/Green style deployments
  provider: istio
  # deployment reference
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: podinfo
  # the maximum time in seconds for the canary deployment
  # to make progress before rollback (default 600s)
  progressDeadlineSeconds: 60
  # HPA reference (optional)
  autoscalerRef:
    apiVersion: autoscaling/v2beta1
    kind: HorizontalPodAutoscaler
    name: podinfo
  service:
    # container port
    port: 8081
    portDiscovery: true
    gateways:
    - public-gateway.istio-system.svc.cluster.local
    # Istio virtual service host names (optional)
    hosts:
    - app.ciwei.com
  canaryAnalysis:
    # schedule interval (default 60s)
    interval: 30s
    # max number of failed checks before rollback
    threshold: 2
    # number of checks to run before rollback
    iterations: 10
    # Prometheus checks based on 
    # http_request_duration_seconds histogram
    metrics:
      - name: request-success-rate
        # minimum req success rate (non 5xx responses)
        # percentage (0-100)
        threshold: 99
        interval: 1m
      - name: request-duration
        # maximum req duration P99
        # milliseconds
        threshold: 500
        interval: 30s
```

kubectl create -f bluegreen.yaml

### 升级镜像版本

模拟升级发布

```java
kubectl -n test set image deployment/podinfo podinfod=demo:2.0
```

访问：app.ciwei.com/walle 模拟流量呀

蓝绿发布你会发现所有请求瞬间都转移到了更新的2.0版本上 如果遇到错误达到一定阀值会回滚

查看发布情况

```java
kubectl -n istio-system logs deployment/flagger -f | jq .msg

"Initialization done! podinfo.test"
"Synced test/podinfo"
"VirtualService podinfo.test updated"
"New revision detected! Scaling up podinfo.test"
"Starting canary analysis for podinfo.test"
"Advance podinfo.test canary iteration 1/10"
"Advance podinfo.test canary iteration 2/10"
"Advance podinfo.test canary iteration 3/10"
"Advance podinfo.test canary iteration 4/10"
"Advance podinfo.test canary iteration 5/10"
"Advance podinfo.test canary iteration 6/10"
"Advance podinfo.test canary iteration 7/10"
"Advance podinfo.test canary iteration 8/10"
"Advance podinfo.test canary iteration 9/10"
"Advance podinfo.test canary iteration 10/10"
"Copying podinfo.test template spec to podinfo-primary.test"
"HorizontalPodAutoscaler podinfo-primary.test updated"
"Halt advancement podinfo-primary.test waiting for rollout to finish: 1 old replicas are pending termination"
"Promotion completed! Scaling down podinfo.test"
```

蓝绿发布也是最适合生产的