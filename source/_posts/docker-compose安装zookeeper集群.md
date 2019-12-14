title: docker-compose安装zookeeper集群
date: 2019-07-17 15:51:33
tags: [docker,zookeeper]
categories: [综合]
---
## Docker-compose Zookeeper Cluster

Zookeeper容器的虚拟化集群，使用Docker-compose构建，通过主机网络进行沟通

<!--more-->

### 目录结构

```bash
├── docker-zk-cluster-down.sh
├── docker-zk-cluster-up.sh
├── zk-01
│   ├── docker-compose.yml
│   └── .env
├── zk-02
│   ├── docker-compose.yml
│   └── .env
└── zk-03
    ├── docker-compose.yml
    └── .env
```

### 各节点说明

| 节点目录名 | 容器名 | client port | follower port | election port |
| ---------- | ------ | ----------- | ------------- | ------------- |
| zk-01      | zk-01  | 2181        | 2888          | 3888          |
| zk-02      | zk-02  | 2182        | 2889          | 3889          |
| zk-03      | zk-03  | 2183        | 2890          | 3890          |

### 文件说明

其中`.env`配置文件为`docker-compose.yml`提供了多个zookeeper的发现服务节点列表

以`zk-01`为例

如果`zk-01`的端口是2181 那么对应的ZOO_SERVERS中2181端口的地址为0.0.0.0 以此类推呀

其他端口连接地址使用宿主机地址就行

```properties
# set args to docker-compose.yml by default
# set zookeeper servers, pattern is `server.x=ip:follower-port:election-port;client:port`,
# such as "server.1=192.168.1.1:2888:3888;2181 server.2=192.168.1.2:2888:3888;2181", 
# `x` is the `ZOO.MY.ID` in docker-compose.yml, multiple server separator by white space.
# now you can overide the ip for server.1 server.2 server.3, here demonstrate in one machine so ip same.
ZOO_SERVERS=server.1=0.0.0.0:2888:3888;2181 server.2=10.11.22.33:2889:3889;2182 server.3=10.11.22.33:2890:3890;2183
```

`dokcer-compose.yml`为docker-compose的配置文件，以`zk-01`举例

```yaml
version: '3'
services:
    zk-01:
        image: zookeeper:3.5.5
        restart: always
        container_name: zk-01
        ports:
            - 2181:2181 # client port
            - 2888:2888 # follower port
            - 3888:3888 # election port
        environment:
            ZOO_MY_ID: 1 # this zookeeper's id, and others zookeeper node distinguishing
            ZOO_SERVERS: ${ZOO_SERVERS} # zookeeper services list
        network_mode: "host"
```

### 使用说明

1. 请确保所布署的 1~3 台服务器网络可以ping通
2. 确保第一台主机的2181\2888\3888端口未占用，第二台主机的2182\2889\3889端口未占用，第三台主机的2183\2890\3890端口未占用
3. 复制zk-01到第一台主机、复制zk-02到第二台主机、复制zk-03到第三台主机
4. 修改zk-01\zk-02\zk-03目录下的.env中的`ZOO_SERVERS`的值，按上述配置要求修改。修改完后的配置应该是集群内通用的，可以scp复制过去。
5. 单台主机请为`docker-zk-cluster-up.sh`与`docker-zk-cluster-down.sh`授执行权，使用它们进行up和down操作；多台主机请手动分别进入zk-0x目录，执行`docker-compose up -d`以启动，执行`docker-compose down`以关闭。