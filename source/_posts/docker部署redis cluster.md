cover: http://ciwei2.cn-sh2.ufileos.com/25.jpg
title: docker部署redis cluster
date: 2019-05-23 17:24:20
tags: [redis]
categories: [综合]
---
### 集群搭建

用的伪集群，真正的集群放到不同的机器即可。端口是7001-7006

工作目录：/data/redis

<!--more-->

### 创建文件夹

首先创建一堆对应端口的文件夹，下面是脚本

create.sh

```java
for i in `seq 7001 7006`
do
 mkdir -p ${i}/data
done
```

添加执行权限并执行

```java
chmod 777 create.sh
./create.sh
```

### 部署redis

创建docker-compose.yml

用publicisworldwide/redis-cluster镜像的原因是人家已经把配置文件写好了，配置文件没有挂载是懒，别学我

docker-compose.yml

```java
version: '3.4'

x-image:
 &default-image
 publicisworldwide/redis-cluster
x-restart:
 &default-restart
 always
x-netmode:
 &default-netmode
 host

services:
 redis1:
  image: *default-image
  network_mode: *default-netmode
  restart: *default-restart
  volumes:
  - /data/redis/7001/data:/data
  environment:
  - REDIS_PORT=7001

 redis2:
  image: *default-image
  network_mode: *default-netmode
  restart: *default-restart
  volumes:
  - /data/redis/7002/data:/data
  environment:
  - REDIS_PORT=7002

 redis3:
  image: *default-image
  network_mode: *default-netmode
  restart: *default-restart
  volumes:
  - /data/redis/7003/data:/data
  environment:
  - REDIS_PORT=7003

 redis4:
  image: *default-image
  network_mode: *default-netmode
  restart: *default-restart
  volumes:
  - /data/redis/7004/data:/data
  environment:
  - REDIS_PORT=7004

 redis5:
  image: *default-image
  network_mode: *default-netmode
  restart: *default-restart
  volumes:
  - /data/redis/7005/data:/data
  environment:
  - REDIS_PORT=7005

 redis6:
  image: *default-image
  network_mode: *default-netmode
  restart: *default-restart
  volumes:
  - /data/redis/7006/data:/data
  environment:
  - REDIS_PORT=7006
```

### 启动所有redis

```java
docker-compose up -d
```

### 部署cluster

运行以下命令(inem0o/redis-trib没有pull会自动pull)

注意：加上-it，不然后续的确认没法继续

```java
docker run --rm -it inem0o/redis-trib create --replicas 1 192.168.30.70:7001 192.168.30.70:7002 192.168.30.70:7003 192.168.30.70:7004 192.168.30.70:7005 192.168.30.70:7006
```

会出现

```java
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.30.70:7001
192.168.30.70:7002
192.168.30.70:7003
Adding replica 192.168.30.70:7004 to 192.168.30.70:7001
Adding replica 192.168.30.70:7005 to 192.168.30.70:7002
Adding replica 192.168.30.70:7006 to 192.168.30.70:7003
M: 5a7bd7698b1fe55beb44faac051d66c8a03fd1b1 192.168.30.70:7001
   slots:0-5460 (5461 slots) master
M: bb8fda08e1dcd39e937443f81b5458e80f52d804 192.168.30.70:7002
   slots:5461-10922 (5462 slots) master
M: d907530ee9f6356e0e61a6c7f4d0cc1b22da1189 192.168.30.70:7003
   slots:10923-16383 (5461 slots) master
S: 52eee69afa751d71c84d5436d14b0e16a37536fa 192.168.30.70:7004
   replicates 5a7bd7698b1fe55beb44faac051d66c8a03fd1b1
S: 701ed2fbb3df9fc63b083818620f5c020d05e323 192.168.30.70:7005
   replicates bb8fda08e1dcd39e937443f81b5458e80f52d804
S: a3548a9dffa225f05786ea2289db65f5f1c623be 192.168.30.70:7006
   replicates d907530ee9f6356e0e61a6c7f4d0cc1b22da1189
Can I set the above configuration? (type 'yes' to accept):
```

输入yes

```java
Waiting for the cluster to join.....
>>> Performing Cluster Check (using node 192.168.30.70:7001)
M: 5a7bd7698b1fe55beb44faac051d66c8a03fd1b1 192.168.30.70:7001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: d907530ee9f6356e0e61a6c7f4d0cc1b22da1189 192.168.30.70:7003@17003
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: a3548a9dffa225f05786ea2289db65f5f1c623be 192.168.30.70:7006@17006
   slots: (0 slots) slave
   replicates d907530ee9f6356e0e61a6c7f4d0cc1b22da1189
S: 701ed2fbb3df9fc63b083818620f5c020d05e323 192.168.30.70:7005@17005
   slots: (0 slots) slave
   replicates bb8fda08e1dcd39e937443f81b5458e80f52d804
S: 52eee69afa751d71c84d5436d14b0e16a37536fa 192.168.30.70:7004@17004
   slots: (0 slots) slave
   replicates 5a7bd7698b1fe55beb44faac051d66c8a03fd1b1
M: bb8fda08e1dcd39e937443f81b5458e80f52d804 192.168.30.70:7002@17002
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

可以使用redis desktop manager 最新版本支持集群 在网盘下载