---
title: Docker 构建脚本 Dockerfile 指令全解析
author: Ciwei
img: ''
coverImg: ''
top: false
cover: false
toc: true
mathjax: false
password: ''
summary: ''
tags:
  - dockerfile
categories:
  - 综合
date: 2019-12-07 17:33:02
---

# Dockerfile基础

<!--more-->

## 1. 前言

**Dockerfile** 是用来构建自定义 **Docker** 镜像的文本文档。我们通过`docker build` 命令用于从**Dockerfile** 文件构建镜像。 如果你要构建自定义镜像，**Dockerfile** 是你必须学会的技能之一。

## 2. Dockerfile 的基本结构

**Dockerfile** 一般分为：基础镜像、镜像元信息、镜像操作指令和容器启动时执行指令，`#` 为 **Dockerfile** 中的注释。

## 3. Dockerfile 文件说明

**Docker** 从上到下的顺序运行**Dockerfile** 的指令，每一个指令都以 `step` 为步骤。而且文件的命名也必须为 `Dockerfile` 。

## 4. Dockerfile 常用的指令。

接下来对常用的 **Dockerfile** 指令进行总结。

### 4.1 FROM 指令

`FROM`是指定基础镜像，必须为第一个命令，格式：

`FROM <image>:<tag>`

其中 `tag` 或 `digest` 是可选的，如果不使用这两个值时，会使用 `latest` 版本的基础镜像。

示例： `FROM mysql:5.6`

### 4.2 ~~MAINTAINER~~ 指令

`MAINTAINER` 用来声明维护者信息,**该命令已经过期**，推荐使用 `LABEL` ，格式：

`MAINTAINER <name>`

### 4.3 LABEL 指令

LABEL：用于为镜像添加元数据,多用于声明构建信息，作者、机构、组织等。格式：

`LABEL <key>=<value> <key>=<value> <key>=<value> ...`

示例： `LABEL version="1.0" description="felord.cn" by="Felordcn"`

使用`LABEL` 指定元数据时，一条LABEL指定可以指定一或多条元数据，指定多条元数据时不同元数据之间通过空格分隔。推荐将所有的元数据通过一条LABEL指令指定，以免生成过多的中间镜像。

### 4.4 ENV 指令

`ENV` 用来设置环境变量，格式：

```bash
ENV <key> <value>  

ENV <key>=<value>
```

示例： `ENV version 1.0.0` 或者 `ENV version=1.0.0`

可以通过 `${key}` 在其它指令中来引用变量，如 `${version}` 。我们也可以通过 `docker run` 中的 `-e <ENV>` 来动态赋值

### 4.5 ARG 指令

`ARG` 用于指定传递给构建运行时的变量，格式：

`ARG <name>[=<default value>]`

通过 `docker run` 中的 `--build-arg <key>=<value>` 来动态赋值，不指定将使用其默认值。

### 4.6 WORKDIR 指令

`WORKDIR` 用来指定工作目录，类似于我们通常使用的`cd` 命令，格式：

`WORKDIR <PATH>`

通过 `WORKDIR` 设置工作目录，**Dockerfile** 中的其它指令 `RUN`、`CMD`、`ENTRYPOINT`、`ADD`、`COPY`等命令都会在该目录下执行。在使用 `docker run` 运行容器时，可以通过 `-w` 参数覆盖构建时所设置的工作目录。

### 4.7 ADD 指令

`ADD` 用于将本地文件添加到镜像中，`tar` 类型文件会自动解压(网络压缩资源不会被解压)，可以访问网络资源，类似 `wget`，格式：

```bash
ADD <src>... <dest>
# 用于支持包含空格的路径
ADD ["<src>",... "<dest>"]
```

示例：

> ADD home* /path/ # 支持通配符 * 添加所有以”home”开头的文件 到/path/ 下

### 4.8 COPY 指令

`COPY` 的功能类似于 `ADD`，但是不会自动解压文件，也不能访问网络资源

### 4.9 RUN 指令

`RUN` 用来执行构建镜像时执行的命令，有以下两种命令执行方式：

- `shell` 执行格式：

`RUN <command>`

示例：`RUN apk update`

- `exec` 执行格式：

    `RUN ["executable", "param1", "param2"]`

示例： `RUN ["/dev/file", "p1", "p2"]`

需要注意的是：　`RUN` 指令创建的中间镜像会被缓存，并会在下次构建中使用。如果不想使用缓存镜像，可在构建时指定 `--no-cache` 参数，示例：`docker build --no-cache`

### 4.10 CMD 指令

`CMD` 构建容器后执行的命令，也就是在容器启动时才执行的命令。格式：

```bash
# 执行可执行文件，优先执行
CMD ["executable","param1","param2"]  
# 设置了 ENTRYPOINT，则直接调用ENTRYPOINT添加参数  参见 CMD 讲解 
CMD ["param1","param2"] 
# 执行shell命令
CMD command param1 param2
```

示例： `CMD ["/usr/bin/bash","--help"]`

`CMD` 不同于 `RUN`，`CMD` 用于指定在容器启动时所要执行的命令，而RUN用于指定镜像构建时所要执行的命令。

### 4.11 ENTRYPOINT 指令

`ENTRYPOINT` 用来配置容器，使其可执行化。配合 `CMD`可省去 `application`，只使用参数。格式：

```bash
#可执行文件, 优先
ENTRYPOINT ["executable", "param1", "param2"]  
# shell内部命令
ENTRYPOINT command param1 param2
```

示例：

```bash
FROM ubuntu

ENTRYPOINT ["top", "-b"]

CMD ["-c"]
```

`ENTRYPOINT` 与 `CMD` 非常类似，不同的是通过 `docker run` 执行的命令不会覆盖 `ENTRYPOINT` ，而 `docker run` 命令中指定的任何参数都会被当做参数再次传递给 `ENTRYPOINT` 指令。**Dockerfile** 中只有最后一个 `ENTRYPOINT` 命令起作用，也就是说如果你指定多个`ENTRYPOINT`,只执行最后的 `ENTRYPOINT` 指令。

### 4.12 EXPOSE 指令

`EXPOSE` 指定与外界交互的端口，格式：

`EXPOSE [<port>...]`

示例: `EXPOSE 8080 443` 、`EXPOSE 80` 、`EXPOSE 11431/tcp 12551/udp`

`EXPOSE` 并不会直接让容器的端口映射主机。宿主机访问容器端口时，需要在 `docker run` 运行容器时通过 `-p` 来发布这些端口，或通过 `-P` 参数来发布`EXPOSE` 导出的所有端口

### 4.13 VOLUME 指令

`VOLUME` 用于指定持久化目录, 格式：

`VOLUME ["<src>",...]`

示例：`VOLUME ["/data"]`，`VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"]`

一个卷可以存在于一个或多个容器的指定目录，该目录可以绕过联合文件系统，并具有以下功能：

1. 卷可以容器间共享和重用
2. 容器并不需要要和其它容器共享卷
3. 修改卷后会立即生效
4. 对卷的修改不会对镜像产生影响
5. 卷会一直存在，直到没有任何容器在使用它

和 `EXPOSE` 指令类似， `VOLUME` 并不会挂载的宿主机，需要通过 `docker run` 运行容器时通过 `-v` 来映射到宿主机的目录中。参见另一个命令 `docker volume create`

### 4.14 USER 指令

`USER` 指定运行容器时的用户名或 `UID`，后续的 `RUN` 也会使用指定用户。使用 `USER` 指定用户时，可以使用用户名、`UID` 或`GID`，或是两者的组合。当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户,格式:

```bash
USER user
USER user:group
USER uid:group
USER uid
USER user:gid
USER uid:gid
```

使用 `USER` 指定用户后，**Dockerfile** 中其后的命令 `RUN`、`CMD`、`ENTRYPOINT` 都将使用该用户。你可以通过 `docker run` 运行容器时，可以通过 `-u` 参数来覆盖指定用户。

### 4.15 ONBUILD 指令

`ONBUILD` 作用是其当所构建的镜像被用做其它镜像的基础镜像，该镜像中的 `ONBUILD` 中的命令就会触发，格式：

`ONBUILD [INSTRUCTION]`

示例：

```bash
ONBUILD ADD . /application/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
```

## 5. 总结

今天对构建 **Docker** 镜像脚本 **Dockerfile** 基本命令进行的详细的总结，并加以举例说明，相信能解决你在构建镜像中的一些困惑。敬请多多关注微信公众号：Felordcn ，后续将会有更多干货奉上。

## 附: Spring Boot Dockerfile

```bash
# 使用 aws 的java jdk 8
FROM amazoncorretto:8
# 作者等相关的元信息
LABEL AUTHOR=Felordcn OG=felord.cn
# 挂载卷 
VOLUME ["/tmp","/logs"]
# 时区
ENV TZ=Asia/Shanghai
# 启用配置文件 默认为 application.yml  
ENV ACTIVE=defualt
# 设置镜像时区
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
# 修改为打包后的jar文件名称
ADD /target/flyway-spring-boot-1.0.0.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-Dspring.profiles.active=${ACTIVE}","-jar","app.jar"]
```