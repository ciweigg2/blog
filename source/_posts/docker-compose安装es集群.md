title: docker-compose安装es集群
date: 2019-07-29 17:48:18
tags: [elasticsearch]
categories: [综合]
---
# docker-es-cluster

本仓库使用docker-compose构建elasticsearch 7.1.0集群

<!--more-->

由于空文件夹提交会被忽略，所以data目录和logs目录下面创建了package.info，使用前需要删除各数据目录下的文件，请保留数据目录和日志目录。

为了对比哪些需要删除，请参考如下目录结构（均需要保留，`.sh`后缀的脚本可以保留）

## 目录结构

```bash
.
├── docker-es-data01
│   ├── data01
│   ├── data01-logs
│   ├── docker-compose.yml
│   ├── .env
│   └── es-config
│       └── elasticsearch.yml
├── docker-es-data02
│   ├── data02
│   ├── data02-logs
│   ├── docker-compose.yml
│   ├── .env
│   └── es-config
│       └── elasticsearch.yml
├── docker-es-data03
│   ├── data03
│   ├── data03-logs
│   ├── docker-compose.yml
│   ├── .env
│   └── es-config
│       └── elasticsearch.yml
├── docker-es-master
│   ├── docker-compose.yml
│   ├── .env
│   ├── es-config
│   │   └── elasticsearch.yml
│   ├── master-data
│   └── master-logs
└── docker-es-tribe
    ├── docker-compose.yml
    ├── .env
    ├── es-config
    │   └── elasticsearch.yml
    ├── tribe-data
    └── tribe-logs
```

## 各目录代表节点与端口号

| 节点目录         | 节点名称 | 协调端口号 | 说明                         | 查询端口号 |
| ---------------- | -------- | ---------- | ---------------------------- | ---------- |
| docker-es-data01 | data01   | 9301       | 数据节点1，非master节点      | 9201       |
| docker-es-data02 | data02   | 9302       | 数据节点2，非master节点      | 9202       |
| docker-es-data03 | data03   | 9303       | 数据节点3，非master节点      | 9203       |
| docker-es-master | master   | 9300       | master节点，非数据节点       | 9200       |
| docker-es-tribe  | tribe    | 9304       | 协调节点，非master非数据节点 | 9204       |

想测试这些节点是否可用，只需要修改**每个**节点目录下的`es-config/elasticsearch.yml`中的ip地址，全部换成你需要的ip即可。

## 各节点操作命令

需要修改sysctl.conf添加vm.max_map_count

```
vi /etc/sysctl.conf
vm.max_map_count=262144
sysctl -p
```

**后台启动命令**均为`docker-compose up -d`

**关闭命令**:

- `docker-compose down`: 关闭同时移除容器与多余虚拟网卡
- `docker stop contains_name`: 根据容器名称关闭容器，不移除容器

## 简单的脚本

**docker-es-cluster-up.sh**

```bash
#/bin/bash
# please put this shell script to the root of each node folder.
# this shell script for start up the docker-es-cluster designed in the one of linux server.
cd docker-es-master && docker-compose up -d && \
cd ../docker-es-data01 && docker-compose up -d && \
cd ../docker-es-data02 && docker-compose up -d && \
cd ../docker-es-data03 && docker-compose up -d && \
cd ../docker-es-tribe && docker-compose up -d && \
cd ..
```

**docker-es-cluster-down.sh**

```bash
#/bin/bash
# please put this shell script to the root of each node folder.
# this shell script for remove the docker-es-cluster's containers and networks designed in the one of linux server.
cd docker-es-tribe && docker-compose down && \
cd ../docker-es-data03 && docker-compose down && \
cd ../docker-es-data02 && docker-compose down && \
cd ../docker-es-data01 && docker-compose down && \
cd ../docker-es-master && docker-compose down && \
cd ..
```

**docker-es-cluster-stop.sh**

```bash
#/bin/bash
# please put this shell script to the root of each node folder.
# this shell script for stop the docker-es-cluster's containers designed in the one of linux server.
docker stop es-tribe es-data03 es-data02 es-data01 es-master
```

> 如果你想让这些脚本有执行权限，不妨试试`sudo chmod +x *.sh`

> 这些脚本中没有使用sudo，如需要使用sudo才能启动docker,请添加当前用户到docker组

> 需要赋予挂载的log和data目录权限为777 `sudo chmod 777 -R 目录`

访问ip：9200/_cat/nodes 查看集群状态

### 各文件功能

鉴于这里边有很多是重复操作，这里仅拿其中的master节点进行举例

`.env` 这个文件为docker-compose.yml提供默认参数，方便修改

```
# the default environment for es-master
# set es node jvm args
ES_JVM_OPTS=-Xms256m -Xmx256m
# set master node data folder
MASTER_DATA_DIR=./master-data
# set master node logs folder
MASTER_LOGS_DIR=./master-logs
```

`docker-compose.yml` docker-compose的配置文件

```
version: "3"
services:
    es-master:
        image: elasticsearch:7.1.0
        container_name: es-master
        environment: # setting container env
            - ES_JAVA_OPTS=${ES_JVM_OPTS}   # set es bootstrap jvm args
        restart: always
        volumes:
            - ./es-config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
            - ${MASTER_DATA_DIR}:/usr/share/elasticsearch/data:rw
            - ${MASTER_LOGS_DIR}:/usr/share/elasticsearch/logs:rw
        network_mode: "host"
```

> 简单来说，就是修改pull的镜像，替换其中的变量与配置文件，挂载数据与日志目录，最后用的host主机模式，让节点服务占用到实体机端口

`elaticsearch.yml` elasticsearch的配置文件，搭建集群最关键的文件之一

```
# ======================== Elasticsearch Configuration =========================
cluster.name: es-cluster
node.name: master 
node.master: true
node.data: false
node.attr.rack: r1 
bootstrap.memory_lock: true 
http.port: 9200
network.host: 10.2.114.110
transport.tcp.port: 9300
discovery.seed_hosts: ["127.0.0.1:9301","127.0.0.1:9302","127.0.0.1:9303","127.0.0.1:9304"] 
cluster.initial_master_nodes: ["master"] 
gateway.recover_after_nodes: 2
```

这里简单说下几个比较重要的参数

* transport.tcp.port 设置es多节点协调的端口号

* discovery.seed_hosts 设置当前节点启动后要发现的协调节点位置，当然自己不需要发现自己，推荐使用ip:port形式，集群形成快

* cluster.initial_master_nodes 集群中可以成为master节点的节点名，这里指定唯一的一个

详细介绍elasticsearch.yml配置文件呀

```
cluster.name: es-cluster #指定es集群名
node.name: xxxx #指定当前es节点名
node.data: false #非数据节点
node.master: false #非master节点
node.attr.rack: r1 #自定义的属性,这是官方文档中自带的
bootstrap.memory_lock: true #开启启动es时锁定内存
network.host: 172.17.0.5 #当前节点的ip地址
http.port: 9200 #设置当前节点占用的端口号，默认9200
discovery.seed_hosts: ["172.17.0.3:9300","172.17.0.4:9300","172.17.0.2:9300"] #启动当前es节点时会去这个ip列表中去发现其他节点，此处不需配置自己节点的ip,这里支持ip和ip:port形式,不加端口号默认使用ip:9300去发现节点
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"] #可作为master节点初始的节点名称,tribe-node不在此列
gateway.recover_after_nodes: 2 #设置集群中N个节点启动时进行数据恢复，默认为1。可选
path.data: /path/to/path  #数据保存目录
path.logs: /path/to/path #日志保存目录
transport.tcp.port: 9300 #设置集群节点发现的端口
```

### 使用说明

1. 若想将此脚本使用到生产上，需要修改每个节点下的.env文件，将挂载数据、日志目录修改为启动es的集群的用户可读写的位置，可以通过`sudo chmod 777 -R 目录` 或 `sudo chown -R 当前用户名:用户组 目录` 来修改被挂载的目录权限
2. 修改`.env`下的JVM参数，扩大堆内存，启动与最大值最好相等，以减少gc次数，提高效率
3. 修改所有节点下的`docker-compose.yml` 中的`network.host`地址 为当前所放置的主机的ip，`discovery.seed_hosts`需要填写具体各待发现节点的实体机ip，以确保可以组成集群
4. 确保各端口在其宿主机上没有被占用，如有占用需确认是否有用，无用kill，有用则更新`docker-compose.yml`的`http.port`或`transport.tcp.port`，注意与此同时要更新其它节点的`discovery.seed_hosts`对应的port
5. 如果在同一台主机上，可以参考使用文章后边的简单的shell脚本

demo：https://github.com/ciweigg2/docker-es-cluster