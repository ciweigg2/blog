cover: http://ciwei2.cn-sh2.ufileos.com/15.jpg
title: docker安装elasticsearch6.8.0
date: 2019-06-25 17:50:25
tags: [elasticsearch]
categories: [综合]
---
### 安装elasticsearch

```java
docker pull elasticsearch:6.8.0
```

<!--more-->

创建network

```java
docker network create dnw_es_680
```

创建容器

```java
docker run -d --name es_680 --net dnw_es_680 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:6.8.0
```

查看ES状态

```java
http://localhost:9200
```

当前集群名为："cluster_name": "docker-cluster"

修改es配置，支持跨域访问

```java
docker exec -it es_680 /bin/bash
vi config/elasticsearch.yml
添加：
	http.cors.enabled: true
	http.cors.allow-origin: "*"
重启容器：
	docker restart es_680
```

安装elasticsearch-head

vi docker-compose.yml

```java
version: '2'
services:
  elasticsearch-head:
      image: wallbase/elasticsearch-head:6-alpine
      container_name: elasticsearch-head
      environment:
        TZ: 'Asia/Shanghai'
      ports:
        - '9100:9100'
```

启动：docker-compose up -d 访问http://127.0.0.1:9100