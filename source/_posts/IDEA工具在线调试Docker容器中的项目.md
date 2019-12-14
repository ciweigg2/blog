cover: http://ciwei2.cn-sh2.ufileos.com/47.jpg
title: IDEA工具在线调试Docker容器中的项目
date: 2019-03-29 17:18:41
tags: [idea,docker]
categories: [综合]
---
### IDEA工具在线调试Docker容器中的项目

<!--more-->

项目根目录创建Dockerfile

```java
FROM frolvlad/alpine-oraclejdk8:slim
VOLUME /tmp
ADD /target/demo-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-jar","-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005","/app.jar"]
```

add docker

![](/images/20190329172440.png)

add remote

![](/images/20190329172517.png)

先起docker 再起remote 可以直接访问远程地址本地调试