title: docker-compose安装elasticsearch7.4.2
date: 2019-11-25 16:54:35
tags: [elasticsearch]
categories: [综合]
---
### docker-compose.yml

<!--more-->

```yml
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.4.2
    container_name: elasticsearch
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - cluster.initial_master_nodes=node-1
      - node.name=node-1
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
    driver: bridge
```

设置配置：

```bash
echo "soft    nproc           4096" > /etc/security/limits.conf
echo "hard    nproc           4096" > /etc/security/limits.conf
echo "vm.max_map_count=262144" > /etc/sysctl.conf
sysctl -p
```

启动elasticsearch：

```bash
docker-compose up -d
```

谷歌elasticsearch可视化插件：https://chrome.google.com/webstore/detail/dejavu-elasticsearch-web/jopjeaiilkcibeohjdmejhoifenbnmlh