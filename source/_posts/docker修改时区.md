cover: http://ciwei2.cn-sh2.ufileos.com/11.jpg
title: docker修改时区
date: 2018-12-22 21:44:36
tags: [docker]
categories: [综合]
---
### docker时区不对 怎么修改

<!--more-->

> 创建容器的时候设置时区

```java
添加 -v /etc/localtime:/etc/localtime 启动容器
docker run -d -v /etc/localtime:/etc/localtime -p 8888:8080 tomcat:latest
```

> 启动后的容器修改时区

```java
docker exec -it <容器名> /bin/bash

ln -sf /usr/share/zoneinfo/Asia/Shanghai    /etc/localtime

docker restart <容器名>
```