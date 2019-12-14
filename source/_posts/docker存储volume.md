cover: http://ciwei2.cn-sh2.ufileos.com/12.jpg
title: docker存储volume
date: 2018-08-15 21:02:36
tags: [docker]
categories: [综合]
---
docker使用volume创建本地数据卷轴
<!--more-->
```
#创建volume
docker volume create volume-test1
#查看参数
docker inspect volume-test1
#使用volume
docker run --name busybox3 -v volume-test1:/volume busybox
#查看
docker inspect -f {{.Mounts}} busybox3
#查看docker数据卷
docker volume ls
#删除没使用的数据卷(谨慎使用)
docker volume prune
#删除一个或多个卷
docker volume rm [OPTIONS]
```
