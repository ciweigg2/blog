cover: http://ciwei2.cn-sh2.ufileos.com/126.jpg
title: traefik保持会话，session亲和性，sticky特性
date: 2019-06-13 15:13:17
tags: [traefik,sticky]
categories: [综合]
---
### 介绍

原理会话粘粘：在客户端第一次发起请求时，反向代理为客户端分配一个服务端，并且将该服务端的地址以SetCookie的形式发送给客户端，这样客户端下一次访问该反向代理时，便会带着这个cookie，里面包含了上一次反向代理分配给该客户端的服务端信息。这种机制是通过一个名为Sticky的插件实现的。而Traefik则集成了与Nginx的Sticky相同功能，并且可以在Kubernetes中方便的开启和配置该特性

解决：认证服务器第一次认证到A POD 第2次访问到B POD导致，认证失效问题，保障一致性

<!--more-->

### service层配置

这个最好别添加，后面会有坑的

添加 traefik.ingress.kubernetes.io/affinity: "true"

```java
apiVersion: v1
kind: Service
metadata:
  annotations:
    traefik.ingress.kubernetes.io/affinity: "true"
```

验证

web请求头里带了Cookie信息 就会出现下面的

![](/images/20190613152121.png)