title: 【Mongodb】Docker安装Mongodb shard集群
date: 2019-10-16 14:16:10
tags: [mongodb,Docker]
categories: [综合]
---
### MongoDB分片介绍

#### 分片机制提供了如下三种优势

* 1.对集群进行抽象，让集群“不可见”

MongoDB自带了一个叫做mongos的专有路由进程。mongos就是掌握统一路口的路由器，其会将客户端发来的请求准确无误的路由到集群中的一个或者一组服务器上，同时会把接收到的响应拼装起来发回到客户端。

* 2.保证集群总是可读写

MongoDB通过多种途径来确保集群的可用性和可靠性。将MongoDB的分片和复制功能结合使用，在确保数据分片到多台服务器的同时，也确保了每分数据都有相应的备份，这样就可以确保有服务器换掉时，其他的从库可以立即接替坏掉的部分继续工作。

* 3.使集群易于扩展

当系统需要更多的空间和资源的时候，MongoDB使我们可以按需方便的扩充系统容量。

<!--more-->

#### 分片集群架构

组件 |说明 |
-|-|-
Config Server |存储集群所有节点、分片数据路由信息。默认需要配置3个Config Server节点 |
Mongos |提供对外应用访问，所有操作均通过mongos执行。一般有多个mongos节点。数据迁移和数据自动平衡 |
Mongod |存储应用数据记录。一般有多个Mongod节点，达到数据分片目的 |

![](/images/1190037-20180106150523846-1535223046.png)

> 分片集群的构造

(1) mongos ：数据路由，和客户端打交道的模块。mongos本身没有任何数据，他也不知道该怎么处理这数据，去找config server

(2) config server：所有存、取数据的方式，所有shard节点的信息，分片功能的一些配置信息。可以理解为真实数据的元数据

(3) shard：真正的数据存储位置，以chunk为单位存数据

Mongos本身并不持久化数据，Sharded cluster所有的元数据都会存储到Config Server，而用户的数据会议分散存储到各个shard。Mongos启动后，会从配置服务器加载元数据，开始提供服务，将用户的请求正确路由到对应的碎片。

#### Mongos的路由功能

当数据写入时，MongoDB Cluster根据分片键设计写入数据

当外部语句发起数据查询时，MongoDB根据数据分布自动路由至指定节点返回数据

### 安装shard集群

创建必要的文件夹并且赋权

```shell
mkdir mongodb-shard
cd mongodb-shard
mkdir db1
mkdir db2
mkdir db3
mkdir db4
mkdir db5
mkdir db6
mkdir cdb1
mkdir cdb2
chmod -R 777 ./*
```

一、创建分片服务 shardsvr

```shell
docker run --name rs1_container1 -d -p 20001:20001 -v $PWD/db1:/data/db1 mongo:latest --shardsvr --replSet "rs1" --port 20001 --bind_ip_all --dbpath /data/db1
docker run --name rs1_container2 -d -p 20002:20001 -v $PWD/db2:/data/db2 mongo:latest --shardsvr --replSet "rs1" --port 20001 --bind_ip_all --dbpath /data/db2
docker run --name rs1_container3 -d -p 20003:20001 -v $PWD/db3:/data/db3 mongo:latest --shrdsvr --replSet "rs1" --port 20001 --bind_ip_all --dbpath /data//db3

docker run --name rs2_container1 -d -p 20011:20001 -v $PWD/db4:/data/db4 mongo:latest --shardsvr --replSet "rs2" --port 20001 --bind_ip_all --dbpath /data/db4
docker run --name rs2_container2 -d -p 20012:20001 -v $PWD/db5:/data/db5 mongo:latest --shardsvr --replSet "rs2" --port 20001 --bind_ip_all --dbpath /data/db5
docker run --name rs2_container3 -d -p 20013:20001 -v $PWD/db6:/data/db6 mongo:latest --shardsvr --replSet "rs2" --port 20001 --bind_ip_all --dbpath /data/db6
```

二、创建配置服务 configsvr

```shell
docker run --name config_container1 -d -p 21001:20001 -v $PWD/cdb1:/data/db mongo:latest --configsvr --replSet "crs" --port 20001 --bind_ip_all --dbpath /data/db
docker run --name config_container2 -d -p 21002:20001 -v $PWD/cdb2:/data/db mongo:latest --configsvr --replSet "crs" --port 20001 --bind_ip_all --dbpath /data/db
```

三、启动路由服务 mongos

192.168.0.15改成机器外网ip或者内网ip

```shell
docker run --name mongos_container1 -d -p 22001:20001 --entrypoint "mongos" mongo:latest --configdb crs/192.168.0.15:21001,192.168.0.15:21002 --port 20001 --bind_ip_all
docker run --name mongos_container2 -d -p 22002:20001 --entrypoint "mongos" mongo:latest --configdb crs/192.168.0.15:21001,192.168.0.15:21002 --port 20001 --bind_ip_all
```

四、初始化分片rs1副本集

192.168.0.15改成机器外网ip或者内网ip

```shell
docker exec -it rs1_container1 /bin/bash
# 任意选择rs1分片的一个副本
mongo --port 20001
# 切换数据库
use admin
# 写配置文件
config = {_id:"rs1",members:[ {_id:0,host:"192.168.0.15:20001"}, {_id:1,host:"192.168.0.15:20002"}, {_id:2,host:"192.168.0.15:20003",arbiterOnly:true} ] }
# 初始化副本集
rs.initiate(config)
# 查看副本集状态
rs.status()
```

五、初始化分片rs2副本集

192.168.0.15改成机器外网ip或者内网ip

```shell
docker exec -it rs2_container1 /bin/bash
# 任意选择rs2分片的一个副本
mongo --port 20001
# 切换数据库
use admin
# 写配置文件
config = {_id:"rs2",members:[ {_id:0,host:"192.168.0.15:20011"}, {_id:1,host:"192.168.0.15:20012"}, {_id:2,host:"192.168.0.15:20013",arbiterOnly:true} ] }
# 初始化副本集
rs.initiate(config)
# 查看副本集状态
rs.status()
```

六、初始化配置服务副本集

192.168.0.15改成机器外网ip或者内网ip

```shell
docker exec -it config_container1 /bin/bash
# 任意选择crs分片的一个副本
mongo --port 20001
# 切换数据库
use admin
# 写配置文件
config = {_id:"crs", configsvr:true, members:[ {_id:0,host:"192.168.0.15:21001"}, {_id:1,host:"192.168.0.15:21002"} ] }
# 初始化副本集
rs.initiate(config)
# 查看副本集状态
rs.status()
```

七、通过mongos添加分片关系到configsvr

192.168.0.15改成机器外网ip或者内网ip

```shell
docker exec -it mongos_container1 /bin/bash
mongo --port 20001
use admin
db.runCommand({addshard:"rs1/192.168.0.15:20001,192.168.0.15:20002,192.168.0.15:20003"})
db.runCommand({addshard:"rs2/192.168.0.15:20011,192.168.0.15:20012,192.168.0.15:20013"})
db.runCommand({listshards:1})
```

八、设置数据库、集合分片

```shell
db.runCommand({enablesharding:"mydb"})
db.runCommand({shardcollection:"mydb.person", key:{id:1, company:1}})
```

九、测试分片结果

```shell
use mydb
for (i =0; i<10000;i++){db.person.save({id:i, company:"baidu"})}
db.person.stats()
```

十、其他

设置一个仲裁者(可忽略)

```shell
rs.addArb("192.168.1.122:27017") 
```

![](/images/20191016144219.png)

### 连接测试

提供Navicat Premium和Springboot连接方式

#### Navicat Premium

Navicat Premium连接mongodb选择随便一台mongos地址的端口连接就行

![](/images/20191016144644.png)

#### SpringBoot

database是刚才上面创建的

```properties
spring.data.mongodb.uri=mongodb://192.168.0.15:22001,192.168.0.15:22002/
spring.data.mongodb.database=mydb
```

demo：https://github.com/ciweigg2/springboot-mongodb