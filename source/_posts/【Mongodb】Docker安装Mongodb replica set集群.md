title: 【Mongodb】Docker安装Mongodb replica set集群
date: 2019-10-16 16:36:08
tags: [mongodb,docker]
categories: [综合]
---
### 创建docker-compose文件

<!--more-->

vi docker-compose.yml

```yaml
version: '2'
services:
  db0:
    image: mongo
    restart: always 
    mem_limit: 4G
    volumes:
      - /workspace/mongo-alpha/db0:/data/db
      - /workspace/mongo-alpha/common:/data/common
    environment:
      TZ: Asia/Shanghai
    ports:
      - "27018:27017"
    command: mongod --replSet rs1
    links:
      - db1
      - db2
      
  db1:
    image: mongo
    restart: always 
    mem_limit: 4G
    volumes:
      - /workspace/mongo-alpha/db1:/data/db
      - /workspace/mongo-alpha/common:/data/common
    environment:
      TZ: Asia/Shanghai
    ports:
      - "27019:27017"
    command: mongod --replSet rs1
    
  db2:
    image: mongo
    restart: always 
    mem_limit: 4G
    volumes:
      - /workspace/mongo-alpha/db2:/data/db
      - /workspace/mongo-alpha/common:/data/common
    environment:
      TZ: Asia/Shanghai
    ports:
      - "27020:27017"
    command: mongod --replSet rs1
```

### 创建容器

在docker-compost.yml的相同路径下，执行docker-compose up -d，这样使用docker ps就可以看到有三个容器启动，选择一个容器进入docker exec -it containerid bash

priority:2 为权重

192.168.0.3 需要改成外网

可以添加选举的机器主要用作投票的呀{"_id": 3, "host": "arbiter001:27017", "arbiterOnly": true}

```yaml
mongo

use admin

config = {
  "_id" : "rs1",
  "members" : [
	  {
		  "_id" : 0,
		  "host" : "192.168.0.3:27018",
		  "priority":2
	  },
	  {
		  "_id" : 1,
		  "host" : "192.168.0.3:27019"
	  },
	  {
		  "_id" : 2,
		  "host" : "192.168.0.3:27020"
	  }
  ]
}
  
rs.initiate(config)
```

此时可以使用rs.status()查看集群信息，rs.isMaster()查看当前节点是否是主节点

代码中连接：

```properties
spring.data.mongodb.uri=mongodb://192.168.0.3:27018,192.168.0.3:27019,192.168.0.3:27020/mydb?slaveOk=true&replicaSet=rs1&write=1&readPreference=secondaryPreferred&connectTimeoutMS=300000
```

navicat premium连接：使用master节点就行了(任意节点也行)

demo：https://github.com/ciweigg2/springboot-mongodb