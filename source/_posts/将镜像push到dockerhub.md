title: 将镜像push到dockerhub
date: 2019-08-11 12:25:42
tags: [dockerhub]
categories: [综合]
---
### 构建镜像

```
docker build -t frankxulei/alibaba-java-docker-demo:1.0 .
```

<!--more-->

dockerhub官网：https://hub.docker.com

### 登录dockerhub

```
docker login
```

### push镜像

```
sudo docker push frankxulei/alibaba-java-docker-demo:1.0
```

用户名必须和镜像名字前缀一样frankxulei否则验证失败