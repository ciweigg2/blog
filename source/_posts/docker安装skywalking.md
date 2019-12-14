cover: http://ciwei2.cn-sh2.ufileos.com/22.jpg
title: docker安装skywalking
date: 2019-05-12 15:31:34
tags: [skywalking]
categories: [综合]
---
### docker安装skywalking

官方：https://github.com/apache/skywalking-docker.git

大神的：https://github.com/JaredTan95/skywalking-docker.git

<!--more-->

### 修改

如果是官方的docker需要设置时间

每个镜像添加TZ: 'Asia/Shanghai'：

```java
services:
  elasticsearch:
    environment:
      TZ: 'Asia/Shanghai'
```

elasticsearch持久化数据

修改elasticsearch添加持久化容器挂载

```java
volumes:
  - ./elasticsearch/data:/usr/share/elasticsearch/data
  - ./elasticsearch/logs:/usr/share/elasticsearch/logs
```

```java
创建数据持久化文件夹目录并赋予权限
mkdir elasticsearch/data
chmod -R 777 elasticsearch
```

### 启动

```java
git clone https://github.com/JaredTan95/skywalking-docker.git
cd skywalking-docker/6.x/docker-compose
docker-compose up -d
http://localhost:8080
账号密码都是admin
```

### 下载

* java运行的集成程序包下载：http://skywalking.apache.org/downloads/

修改：H:\apache-skywalking-apm-incubating\agent\config\agent.config collector.backend_service地址

### 项目中集成

> war or jar集成skywalking

- Linux Tomcat 7, Tomcat 8  
修改 `tomcat/bin/catalina.sh`,在首行加入如下信息.
```shell
CATALINA_OPTS="$CATALINA_OPTS -javaagent:/path/to/skywalking-agent/skywalking-agent.jar=agent.service_name=Dubbo"; export CATALINA_OPTS
```
- Windows Tomcat 7, Tomcat 8  
修改 `tomcat/bin/catalina.bat`,在首行加入如下信息.
```shell
set "CATALINA_OPTS=-javaagent:/path/to/skywalking-agent/skywalking-agent.jar=agent.service_name=Dubbo"
```
- JAR file  
在启动你的应用程序的命令行中添加 `-javaagent` 参数. 并确保在`-jar`参数之前添加它. 例如:
 ```shell
-javaagent:H:/apache-skywalking-apm-incubating/agent/skywalking-agent.jar=agent.service_name=Dubbo
例如：
java -javaagent:H:/apache-skywalking-apm-incubating/agent/skywalking-agent.jar=agent.service_name=Dubbo -jar dubbo.jar
多个参数配置使用逗号连接：
java -javaagent:H:/apache-skywalking-apm-bin/agent/skywalking-agent.jar=agent.service_name=Dubbo-Spring-Boot,collector.backend_service=192.168.15.59:11800 -jar dubbo.jar
 ```
 
> 自定义探针配置路径

自定义的探针配置文件内容格式必须与默认探针配置文件内容格式一致，这里所改变的仅仅只是配置文件的路径

**使用方式：使用 `启动参数(-D)` 的方式来设置探针配置文件路径**

```java
-Dskywalking_config=/path/to/agent.config
```

其中的`/path/to/agent.config` 代表的是自定义探针配置文件的绝对路径 

demo：https://github.com/ciweigg2/springboot-dubbo-seata

![](/images/20190512153751.png)