cover: http://ciwei2.cn-sh2.ufileos.com/23.jpg
title: docker安装最新版elasticsearch
date: 2018-09-01 17:38:05
tags: [elasticsearch]
categories: [综合]
---
ELK所有的最新docker版本都在这里(这里使用6.4.0当前最新版本)
最新docker版本的地址：https://www.docker.elastic.co/
<!--more-->
参考：https://www.elastic.co/guide/en/elasticsearch/reference/6.4/docker.html

方式1 没有把数据挂载到宿主机
```
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.4.0
docker run -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" --name elas docker.elastic.co/elasticsearch/elasticsearch:6.4.0
开启跨域权限
docker exec -ti elas bash
echo "http.cors.enabled : true" >> /usr/share/elasticsearch/config/elasticsearch.yml
echo "http.cors.allow-origin : \"*\"" >> /usr/share/elasticsearch/config/elasticsearch.yml
docker restart elas #重启elas容器
```

方式2 挂载数据到宿主机(单机方式)
docker-compose.yml

```
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.4.0
    container_name: elasticsearch
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - http.cors.enabled=true
      - http.cors.allow-origin=*
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9300:9300
    networks:
      - esnet

volumes:
  esdata1:
    driver: local

networks:
  esnet:
```

方式3 集群方式

```
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.4.0
    container_name: elasticsearch
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - "discovery.zen.ping.unicast.hosts=elasticsearch,elasticsearch2"
      - node.name=es1
      - network.host=0.0.0.0
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 1g    
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9300:9300
    networks:
      - esnet
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.4.0
    container_name: elasticsearch2
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - "discovery.zen.ping.unicast.hosts=elasticsearch,elasticsearch2"
      - node.name=es2
      - network.host=0.0.0.0
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 1g    
    volumes:
      - esdata2:/usr/share/elasticsearch/data
    networks:
      - esnet

volumes:
  esdata1:
    driver: local
  esdata2:
    driver: local

networks:
  esnet:
```

> 备注：
discovery.zen.ping.unicast.hosts这个参数是集群机器间的通讯地址 如果要部署多台机器的话要改hostname
一台机器部署一个elasticsearch
使用hostname关联进行通信，每台机器都要修改hostname添加所有机器的地址
192.168.0.1 elasticsearch
192.168.0.2 elasticsearch2

物理主机集群部署方式：https://my.oschina.net/u/3707404/blog/1929638

* 检查集群状态：curl http://127.0.0.1:9200/_cat/health

* 检查集群节点：http://127.0.0.:9200/_cluster/state?pretty (关闭一台后会发现nodes消失了，但是数据依然可以查询到)

启动
docker-compose up -d
暂停
docker-compose down -v

手动添加索引方式：curl -X PUT http://localhost:9200/test
新增了test索引