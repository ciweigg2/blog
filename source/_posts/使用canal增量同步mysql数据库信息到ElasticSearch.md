cover: http://ciwei2.cn-sh2.ufileos.com/136.jpg
title: 使用canal增量同步mysql数据库信息到ElasticSearch
date: 2019-06-24 17:08:41
tags: [canal,mysql,elasticsearch]
categories: [综合]
---
> 本文介绍如何使用canal增量同步mysql数据库信息到ElasticSearch。（注意：是增量）

### 简介

canal介绍

Canal是一个基于MySQL二进制日志的高性能数据同步系统。Canal广泛用于阿里巴巴集团（包括https://www.taobao.com）

以提供可靠的低延迟增量数据管道，github地址：https://github.com/alibaba/canal

Canal Server能够解析MySQL binlog并订阅数据更改，而Canal Client可以实现将更改广播到任何地方，例如数据库和Apache Kafka

<!--more-->

它具有以下功能：

* 支持所有平台
* 支持由Prometheus提供支持的细粒度系统监控
* 支持通过不同方式解析和订阅MySQL binlog，例如通过GTID
* 支持高性能，实时数据同步。（详见Performance）
* Canal Server和Canal Client都支持HA / Scalability，由Apache ZooKeeper提供支持
* Docker支持

缺点：

不支持全量更新，只支持增量更新

完整wiki地址：https://github.com/alibaba/canal/wiki

### 运作原理

原理很简单：

* Canal模拟MySQL的slave的交互协议，伪装成mysql slave，并将转发协议发送到MySQL Master服务器
* MySQL Master接收到转储请求并开始将二进制日志推送到slave（即canal）
* Canal将二进制日志对象解析为自己的数据类型（原始字节流）

如图所示：

![](/images/9953332-6207f5f99059bd7a.jpeg)

### 同步es

在同步数据到es的时候需要使用适配器：canal adapter。目前最新版本1.1.3，下载地址：https://github.com/alibaba/canal/releases。

目前es支持6.x版本，不支持7.x版本

### 准备工作

踩过的坑：如果需要更换mysql 需要在canal.deployer-1.1.3/conf/example删除h2.mv.db和meta.dat

### 安装jdk

参考：linux安装openjdk

### 安装mysql

安装

```java
docker run --name mysql -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -d mysql:5.7
```

修改远程登录(mysql8.0)

```java
docker exec -it mysql /bin/bash
mysql -uroot -p123456
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';
flush privileges;
```

mysql需要开启binlog（mysql8.0默认是开启的，但是canal暂时不支持mysql8.0）

```java
进入mysql容器: 
docker exec -it mysql /bin/bash
进入配置文件目录下: 
cd /etc/mysql/mysql.conf.d/
开启binlog: 
echo 'log-bin=/var/lib/mysql/mysql-bin' >> mysqld.cnf
echo 'server-id=123454' >> mysqld.cnf
修改binlog格式: 
echo 'binlog-format=ROW' >> mysqld.cnf
重启mysql：
docker restart mysql
```

### 安装elasticsearch

```java
docker pull elasticsearch:6.8.0
```

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

### 安装canal server

下载canal.deployer-1.1.3.tar.gz

```java
mkdir -p canal.deployer-1.1.3
cd canal.deployer-1.1.3
wget https://github.com/alibaba/canal/releases/download/canal-1.1.3/canal.deployer-1.1.3.tar.gz
```

解压文件

```java
tar -zxvf canal.deployer-1.1.3.tar.gz
```

进入解压后的文件夹

```java
cd canal.deployer-1.1.3
```

修改conf/example/instance.properties文件，主要注意以下几处：

* canal.instance.master.address：数据库地址，例如127.0.0.1:3306
* canal.instance.dbUsername：数据库用户
* canal.instance.dbPassword：数据库密码

完整内容如下：

```java
#################################################
## mysql serverId , v1.0.26+ will autoGen
# canal.instance.mysql.slaveId=0

# enable gtid use true/false
canal.instance.gtidon=false

# position info
canal.instance.master.address=127.0.0.1:3306
canal.instance.master.journal.name=
canal.instance.master.position=
canal.instance.master.timestamp=
canal.instance.master.gtid=

# rds oss binlog
canal.instance.rds.accesskey=
canal.instance.rds.secretkey=
canal.instance.rds.instanceId=

# table meta tsdb info
canal.instance.tsdb.enable=true
#canal.instance.tsdb.url=
#canal.instance.tsdb.dbUsername=
#canal.instance.tsdb.dbPassword=

#canal.instance.standby.address =
#canal.instance.standby.journal.name =
#canal.instance.standby.position =
#canal.instance.standby.timestamp =
#canal.instance.standby.gtid=

# username/password
canal.instance.dbUsername=root
canal.instance.dbPassword=12345678
canal.instance.connectionCharset = UTF-8
# enable druid Decrypt database password
canal.instance.enableDruid=false
#canal.instance.pwdPublicKey=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALK4BUxdDltRRE5/zXpVEVPUgunvscYFtEip3pmLlhrWpacX7y7GCMo2/JM6LeHmiiNdH1FWgGCpUfircSwlWKUCAwEAAQ==

# table regex
canal.instance.filter.regex=.*\\..*
# table black regex
canal.instance.filter.black.regex=

# mq config
#canal.mq.topic=example
# dynamic topic route by schema or table regex
#canal.mq.dynamicTopic=mytest1.user,mytest2\\..*,.*\\..*
#canal.mq.partition=0
# hash partition config
#canal.mq.partitionsNum=3
#canal.mq.partitionHash=test.table:id^name,.*\\..*
#################################################
```

回到canal.deployer-1.1.3目录下，启动canal：

```java
sh bin/startup.sh
```

查看日志：

```java
tail -f logs/canal/canal.log
```

查看具体instance日志：

```java
tail -f logs/example/example.log
```

关闭命令

```java
sh bin/stop.sh
```

### 安装canal-adapter

下载canal.adapter-1.1.3.tar.gz

```java
mkdir -p canal.adapter-1.1.3
cd canal.adapter-1.1.3
wget https://github.com/alibaba/canal/releases/download/canal-1.1.3/canal.adapter-1.1.3.tar.gz
```

解压

```java
tar -zxvf canal.adapter-1.1.3.tar.gz
```

进入解压后的文件夹

```java
cd canal.adapter-1.1.3
```

修改conf/application.yml文件，主要注意如下内容，由于是yml文件，注意我这里说明的属性名称：

```java
server.port:canal-adapter端口号
canal.conf.canalServerHost:canal-server地址和ip
canal.conf.srcDataSources.defaultDS.url:数据库地址
canal.conf.srcDataSources.defaultDS.username:数据库用户名
canal.conf.srcDataSources.defaultDS.password:数据库密码
canal.conf.canalAdapters.groups.outerAdapters.hosts:es主机地址,tcp端口
canal.conf.canalAdapters.groups.outerAdapters.properties:cluster.name:集群名
```

另外需要配置conf/es/*.yml文件，adapter将会自动加载conf / es下的所有.yml结尾的配置文件。在介绍配置前，需要先介绍一下本案例使用的表结构，如下：

首先创建数据库test

```java
CREATE TABLE `test` (
  `id` int(11) NOT NULL,
  `name` varchar(200) NOT NULL,
  `address` varchar(1000) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

需要手动去es中创建索引，比如这里使用es-head创建，如下图：

![](/images/9953332-9010dceccfe568eb.jpg)

test索引结构如下：

```java
{
    "mappings":{
        "_doc":{
            "properties":{
                "name":{
                    "type":"text"
                },
                "address":{
                    "type":"text"
                }
            }
        }
    }
}
```

接下来创建test.yml（文件名随意），内容很好理解_index为索引名称，sql为对应语句，内容如下：

```java
dataSourceKey: defaultDS
destination: example
groupId:
esMapping:
  _index: test
  _type: _doc
  _id: _id
  upsert: true
  sql: "select a.id as _id,a.name,a.address from test a"
  commitBatch: 3000
```

配置完成后，回到canal-adapter根目录，执行命令启动

```java
bin/startup.sh
```

查看日志

```java
tail -f logs/adapter/adapter.log
```

关闭canal-adapter命令

```java
bin/stop.sh
```

### 测试

都启动成功后，先查看一下es-head，如图，现在是没有任何数据的

![](/images/20190624.jpg)

接下来，我们在数据库中插入一条数据进行测试，语句如下：

```java
INSERT INTO `test`.`test`(`id`, `name`, `address`) VALUES (7, '北京', '北京市朝阳区');
```

然后在看一下es-head，如下

![](/images/201906241.jpg)

接下来看一下日志，如下：

```java
2019-06-22 17:54:15.385 [pool-2-thread-1] DEBUG c.a.otter.canal.client.adapter.es.service.ESSyncService - DML: {"data":[{"id":7,"name":"北京","address":"北京市朝阳区"}],"database":"test","destination":"example","es":1561197255000,"groupId":null,"isDdl":false,"old":null,"pkNames":["id"],"sql":"","table":"test","ts":1561197255384,"type":"INSERT"} 
Affected indexes: test
```

查看最后200行日志呀

```java
tail -200f logs/adapter/adapter.log
```

增删改查都会影响es中的数据呀