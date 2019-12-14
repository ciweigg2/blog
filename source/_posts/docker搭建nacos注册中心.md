cover: http://ciwei2.cn-sh2.ufileos.com/24.jpg
title: docker搭建nacos注册中心
date: 2019-01-13 13:03:16
tags: [nacos]
categories: [nacos]
---
### 操作步骤

* Clone 项目
```java
git clone https://github.com/nacos-group/nacos-docker.git
cd nacos-docker
```

<!--more-->

* 单机模式
```java
docker-compose -f example/standalone-derby.yaml up -d
```

* 单机模式mysql
```java
docker-compose -f example/standalone-mysql.yaml up -d
```

* 集群模式
```java
docker-compose -f example/cluster-hostname.yaml up -d
```

* 服务注册
```java
curl -X PUT 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=nacos.naming.serviceName&ip=20.18.7.10&port=8080'
```

* 服务发现
```java
curl -X GET 'http://127.0.0.1:8848/nacos/v1/ns/instances?serviceName=nacos.naming.serviceName'
```

* 发布配置
```java
curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test&content=helloWorld"
```

* 获取配置
```java
  curl -X GET "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test"
```
  
* Nacos 控制台
```java
link：http://127.0.0.1:8848/nacos/
```