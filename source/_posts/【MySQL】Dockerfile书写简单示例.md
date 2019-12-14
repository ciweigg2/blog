title: 【MySQL】Dockerfile书写简单示例
date: 2019-07-22 15:14:49
tags: [mysql,docker]
categories: [mysql]
---
**前言：**在用MySQL镜像启动MySQL服务时，我们经常会遇到时区和字符集问题，本篇文章将以Dockerfile形式重新构建MySQL镜像来彻底解决此问题。

<!--more-->

##### 1.拉取官方镜像

```shell
docker pull mysql:5.7.17
```

##### 2.创建dockerfile

```shell
mkdir mysqldb
cd mysqldb
vi Dockerfile
```

构建一个 Dockerfile 文件内容为：

```shell
FROM mysql:5.7.17
MAINTAINER wang
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
ENV LANG=C.UTF-8
```

##### 3.重新构建镜像

在 Dockerfile 文件所在目录执行：

```shell
docker build -t mysqldb:5.7.17 .
```

命令最后有一个. 表示当前目录

构建完成之后，使用`docker images`检查镜像

接下来使用 docker run 命令来启动容器 核实时区及字符集是否正确

```shell
docker run --name mysqldb -e MYSQL_ROOT_PASSWORD=yourpass -d mysqldb:5.7.17
```

