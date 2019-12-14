cover: http://ciwei2.cn-sh2.ufileos.com/140.jpg
title: 使用Trampoline在本地管理Spring Boot应用程序
date: 2019-01-06 14:30:31
tags: [springboot,Trampoline]
categories: [可视化工具]
---
### Trampoline管理springboot应用

> 介绍

http://www.spring4all.com/article/3318

Trampoline可以管理springcloud springboot-dubbo微服务和单个springboot服务

可以配置springcloud微服务的启动顺序 延迟启动 可以重新启动微服务kill服务等功能

<!--more-->

### 安装

源码地址：https://github.com/ErnestOrt/Trampoline.git

```java
git clone https://github.com/ErnestOrt/Trampoline.git
cd Trampoline/trampoline
mvn spring-boot:run
http://localhost:8080
```

![](/images/20190106143610.png)

### 部署项目

> 首先创建springboot项目 actuator可以获取一些信息和控制程序关系开启

```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

```java
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>pl.project13.maven</groupId>
                <artifactId>git-commit-id-plugin</artifactId>
                <version>2.2.5</version>
            </plugin>
        </plugins>
    </build>
```

> 配置application.properties修改port

```java
server.port=8081
```

> 配置application.yml

里面的logging.file配置日志文件是为了界面输出日志

base-path修改actuator的默认路径 默认路径为/actuator 可以不修改

```java
management:
  endpoint:
    shutdown:
      enabled: true
  endpoints:
    web:
      exposure:
        include: '*'
      base-path: /actuator
logging:
  file: trampoline.log
```

### 界面配置springboot微服务

> 按照图片的配置 配置maven(Maven Settings)  和微服务(Register Microservice)

![](/images/20190106144930.jpg)

### 最牛逼的微服务一键部署功能适合分布式项目springcloud 和 springboot-dubbo

> 选中2个以上微服务一起部署 延迟部署

![](/images/20190106145137.png)

> Start at (sec) 微服务启动的等待时间 第一个直接启动 第二个等待5秒后启动 	Launching Order 启动顺序

![](/images/20190106145242.png)

### 启动springboot微服务

* 项目必须上传gitlab github 等
* 支持启动一个springboot多个端口启动 集群很棒哦(这边端口会覆盖项配置目的端口的)
* 可以多台服务器都部署Trampoline 最后用nginx把所有服务集中起来负载均衡的

![](/images/20190106150244.jpg)

### 高级功能监控远程已经启动的springboot微服务

![](/images/20190106151910.jpg)

### 添加远程git仓库的springboot微服务

![](/images/添加远程gitrepo的项目.png)

> 更新代码并重启微服务 但是好像没效果(新版本可能修复了) 实现不行可以到项目路径 git pull

![](/images/20190106160506.png)

测试demo地址：https://github.com/ciweigg2/trampoline-demo.git