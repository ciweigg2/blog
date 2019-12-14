title: docker-compose安装redis sentinel集群
date: 2019-06-10 16:59:24
tags: [redis,sentinel]
categories: [综合]
---
### redis sentinel集群

参考：https://blog.csdn.net/song_java/article/details/88039575

<!--more-->

### 创建目录

```
mkdir -p /home/sentinel/build
```

### 新增docker-compose.yml

需要将外网的ip替换成自己的外网ip

```
cd /home/sentinel
vi docker-compose.yml

version: '2'
services:
  repl-0:
    image: redis:5.0-rc-alpine
    restart: always
    volumes:
      - "/etc/timezone:/etc/timezone:ro"
      - "/etc/localtime:/etc/localtime:ro"
    ports:
      - 6380:6379       
    networks:
      - sentinel
    command: ["redis-server","--replica-announce-ip","外网的ip","--replica-announce-port","6380"]
 
  repl-1:
    image: redis:5.0-rc-alpine
    restart: always
    volumes:
      - "/etc/timezone:/etc/timezone:ro"
      - "/etc/localtime:/etc/localtime:ro"
    ports:
      - 6381:6379       
    networks:
      - sentinel
    command: ["redis-server","--slaveof","repl-0","6379","--replica-announce-ip","外网的ip","--replica-announce-port","6381"]
 
  repl-2:
    image: redis:5.0-rc-alpine
    restart: always
    volumes:
      - "/etc/timezone:/etc/timezone:ro"
      - "/etc/localtime:/etc/localtime:ro"
    ports:
      - 6382:6379       
    networks:
      - sentinel
    command: ["redis-server","--slaveof","repl-0","6379","--replica-announce-ip","外网的ip","--replica-announce-port","6382"]
 
  repl-3:
    image: redis:5.0-rc-alpine
    restart: always
    volumes:
      - "/etc/timezone:/etc/timezone:ro"
      - "/etc/localtime:/etc/localtime:ro"
    ports:
      - 6383:6379       
    networks:
      - sentinel
    ports:
      - 6383:6379
    command: ["redis-server","--slaveof","repl-2","6379","--replica-announce-ip","外网的ip","--replica-announce-port","6383"]
    
  sentinel-1:
    build: ./build
    restart: always
    volumes:
      - "/etc/timezone:/etc/timezone:ro"
      - "/etc/localtime:/etc/localtime:ro"
    networks:
      - sentinel 
    ports:
      - 26380:26379
    command: ["redis-server","/data/sentinel.conf","--sentinel","announce-ip","外网的ip","--sentinel","announce-port" , "26380"]
 
  sentinel-2:
    build: ./build
    restart: always
    volumes:
      - "/etc/timezone:/etc/timezone:ro"
      - "/etc/localtime:/etc/localtime:ro"
    networks:
      - sentinel 
    ports:
      - 26381:26379
    command: ["redis-server","/data/sentinel.conf","--sentinel","announce-ip","外网的ip","--sentinel","announce-port" , "26381"]
    
  sentinel-3:
    build: ./build
    restart: always
    volumes:
      - "/etc/timezone:/etc/timezone:ro"
      - "/etc/localtime:/etc/localtime:ro"
    networks:
      - sentinel  
    ports:
      - 26382:26379
    command: ["redis-server","/data/sentinel.conf","--sentinel","announce-ip","外网的ip","--sentinel","announce-port" , "26382"]
 
networks:
  sentinel: 
     driver: bridge
```

注：与非docker环境相比,我们添加了announce-ip和announce-port这些命令,这是因为默认情况下主从服务器之间以及各个sentinel实例之间使用的ip是容器内部自动分配的ip,这样在切换主从服务或sentinel之间通信时无法联通，因此务必需要添加对应的命令

### 创建 Dockerfile和sentinel.conf

```
cd /home/sentinel/build
```

创建Dockerfile

```
FROM redis:5.0-rc-alpine
COPY ./sentinel.conf /data/sentinel.conf
EXPOSE 26379
```

创建sentinel.conf

```
sentinel monitor mymaster 外网的ip 6380 2
sentinel down-after-milliseconds mymaster 10000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1
```

启动

```
cd /home/sentinel
docker-compose up -d
docker-compose logs -f
```

程序连接：

及群名:mymaster

外网ip:26380

外网ip:26381

外网ip:26382

也可以使用redis desktop manager 连接测试的