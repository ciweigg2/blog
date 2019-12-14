cover: http://ciwei2.cn-sh2.ufileos.com/127.jpg
title: traefik自动熔断
date: 2019-06-13 15:31:35
tags: [traefik]
categories: [综合]
---
### 自动熔断

在集群中，当某一个服务大量出现请求错误，或者请求响应时间过久，或者返回500+错误状态码时，我们希望可以主动剔除该服务，也就是不在将请求转发到该服务上，而这一个过程是自动完成，不需要人工执行。Traefik 通过配置很容易就能帮我们实现，Traefik 可以通过定义策略来主动熔断服务

<!--more-->

举例：

* NetworkErrorRatio() > 0.5：监测服务错误率达到50%时，熔断
* LatencyAtQuantileMS(50.0) > 50：监测延时大于50ms时，熔断
* ResponseCodeRatio(500, 600, 0, 600) > 0.5：监测返回状态码为[500-600]在[0-600]区间占比超过50%时，熔断

### 例子

```java
apiVersion: v1
kind: Service
metadata:
  name: wensleydale
  annotations:
    traefik.backend.circuitbreaker: "NetworkErrorRatio() > 0.5" 
    traefik.backend.circuitbreaker: LatencyAtQuantileMS(50.0) > 2000 #>2秒熔断
```