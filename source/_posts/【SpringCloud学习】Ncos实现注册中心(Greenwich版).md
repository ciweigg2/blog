title: 【SpringCloud学习】Ncos实现注册中心(Greenwich版)
date: 2019-07-20 18:23:32
tags: [springcloud,nacos]
categories: [springcloud]
---
### 添加依赖

添加nacos的springcloud依赖呀

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>0.9.0.RELEASE</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

<!--more-->

### application.properties添加注册中心

```properties
spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
```

### 启动springcloud

发现已经注册到注册中心啦

![](/images/20190720182909.png)

demo：https://github.com/ciweigg2/spring-cloud-examples/tree/spring-cloud-nacos/spring-cloud-clientA