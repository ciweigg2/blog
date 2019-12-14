cover: http://ciwei2.cn-sh2.ufileos.com/20.jpg
title: Docker安装分布式追踪APM工具Pinpoint和项目使用
date: 2018-08-01 16:54:44
tags: [pinpoint]
categories: [综合]
top: true
---
Docker安装Pinpoint，分布式跟踪工具Pinpoint，监控dubbo等一些普通程序
<!--more-->

![](/images/20190521153148.png)

![](/images/20190521153120.png)

```java
克隆官方提供的docker git
git clone https://github.com/naver/pinpoint-docker.git
cd pinpoint-docker
如需修改相关组件的ip和端口，请修改pinpoint-Docker/.env文件
docker-compose pull && docker-compose up -d
访问：http://localhost:8079/ 即可浏览pinpoint页面
官方说明：https://github.com/naver/pinpoint-docker
```

安装Agent组件
 访问 https://github.com/naver/pinpoint/releases 下载和 Collector组件 相同版本的pinpoint-agent压缩包。

 解压pinpoint-agent压缩包，找到 pinpoint.config 文件，修改为你对应环境的配置。一般情况只需要修改配置项 profiler.collector.ip=127.0.0.1 为你自己的Collector组件的IP
 
配置应用程序
* 如果你在多台机器上部署了应用程序，那么就需要在多台机器上部署Agent组件
* ${pinpointPath}是agent组件存放的路径，类似于JAVA_HOME
* 在java启动命令中加入如下参数
* -javaagent:${pinpointPath}/pinpoint-bootstrap-1.7.3.jar
* -Dpinpoint.applicationName=  // 在pinpoint上显示的名字
* -Dpinpoint.agentId=               // id,可以和applicationName相同，也可以不同，全局唯一 
 
springboot启动项目：
```java
java -javaagent:D:\pinpoint\spring-boot-dubbo\pinpoint-agent-1.7.1\pinpoint-bootstrap-1.7.1.jar -Dpinpoint.agentId=ciwei-provider -Dpinpoint.applicationName=ciwei-provider -jar dubbo-provider-1.0-SNAPSHOT.jar

java -javaagent:D:\pinpoint\spring-boot-dubbo\pinpoint-agent-1.7.1\pinpoint-bootstrap-1.7.1.jar -Dpinpoint.agentId=ciwei-consumer -Dpinpoint.applicationName=ciwei-consumer -jar dubbo-consumer-1.0-SNAPSHOT.jar
```
