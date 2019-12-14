cover: http://ciwei2.cn-sh2.ufileos.com/108.jpg
title: springboot-dubbo集成zipkin
date: 2019-01-17 18:06:15
tags: [dubbo,zipkin]
categories: [综合]
---
### 介绍

随着业务的发展，应用的规模不断的扩大，传统的应用架构无法满足诉求，服务化架构改造势在必行，以 Dubbo 为代表的分布式服务框架成为了服务化改造架构中的基石。随着微服务理念逐渐被大众接受，应用进一步向更细粒度拆分，并且，不同的应用由不同的开发团队独立负责，整个分布式系统变得十分复杂。没有人能够清晰及时的知道当前系统整体的依赖关系。当出现问题时，也无法及时知道具体是链路上的哪个环节出了问题。

在这个背景下，Google 发表了 Dapper 的论文，描述了如何通过一个分布式追踪系统解决上述问题。基于该论文，各大互联网公司实现并部署了自己的分布式追踪系统，其中比较出名的有阿里巴巴的 EagleEye。本文中提到的 Zipkin 是 Twitter 公司开源的分布式追踪系统。下面会详细介绍如何在 Dubbo 中使用 Zipkin 来实现分布式追踪。

<!--more-->

文章地址：http://dubbo.apache.org/zh-cn/blog/use-zipkin-in-dubbo.html

### 安装zipkin

* 安装zipkin mysql 方式

参考：https://github.com/openzipkin/docker-zipkin/blob/master/docker-compose.yml

参考：https://github.com/openzipkin/docker-zipkin

```java
vi docker-compose.yml
docker-compose up -d
```

```java
version: '2'

services:
  storage:
    image: openzipkin/zipkin-mysql
    container_name: mysql
    # Uncomment to expose the storage port for testing
    ports:
      - 3309:3306
    volumes:
      - dbfiles:/mysql/data
    restart: unless-stopped

  zipkin:
    image: openzipkin/zipkin
    container_name: zipkin
    # Environment settings are defined here https://github.com/openzipkin/zipkin/tree/1.19.0/zipkin-server#environment-variables
    environment:
      - STORAGE_TYPE=mysql
      # Point the zipkin at the storage backend
      - MYSQL_HOST=mysql
      # Uncomment to enable scribe
      # - SCRIBE_ENABLED=true
      # Uncomment to enable self-tracing
      # - SELF_TRACING_ENABLED=true
      # Uncomment to enable debug logging
      # - JAVA_OPTS=-Dlogging.level.zipkin=DEBUG
    ports:
      # Port used for the Zipkin UI and HTTP Api
      - 9411:9411
    depends_on:
      - storage
    restart: unless-stopped

  dependencies:
    image: openzipkin/zipkin-dependencies
    container_name: dependencies
    entrypoint: crond -f
    environment:
      - STORAGE_TYPE=mysql
      - MYSQL_HOST=mysql
      # Add the baked-in username and password for the zipkin-mysql image
      - MYSQL_USER=zipkin
      - MYSQL_PASS=zipkin
      # Uncomment to adjust memory used by the dependencies job
      - JAVA_OPTS=-verbose:gc -Xms512m -Xmx512m
    depends_on:
      - storage
    restart: unless-stopped

volumes:
  dbfiles:
```

### 集成springboot-dubbo项目中

搭建springboot-dubbo项目上一篇文章讲过了

#### 引入 Brave 依赖
```java
    <properties>
        <brave.version>5.4.2</brave.version>
        <zipkin-reporter.version>2.7.9</zipkin-reporter.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <!-- 引入 zipkin brave 的 BOM 文件 -->
            <dependency>
                <groupId>io.zipkin.brave</groupId>
                <artifactId>brave-bom</artifactId>
                <version>${brave.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
			
            <!-- 引入 zipkin repoter 的 BOM 文件 -->
            <dependency>
                <groupId>io.zipkin.reporter2</groupId>
                <artifactId>zipkin-reporter-bom</artifactId>
                <version>${zipkin-reporter.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

#### 所有需要链路的提供者消费者模块都引入brave依赖
```java
    <dependencies>
        <!-- 1. brave 对 dubbo 的集成 -->
        <dependency>
            <groupId>io.zipkin.brave</groupId>
            <artifactId>brave-instrumentation-dubbo-rpc</artifactId>
        </dependency>

        <!-- 2. brave 的 spring bean 支持 -->
        <dependency>
            <groupId>io.zipkin.brave</groupId>
            <artifactId>brave-spring-beans</artifactId>
        </dependency>

        <!-- 3. 在 SLF4J 的 MDC (Mapped Diagnostic Context) 中支持 traceId 和 spanId -->
        <dependency>
            <groupId>io.zipkin.brave</groupId>
            <artifactId>brave-context-slf4j</artifactId>
        </dependency>

        <!-- 4. 使用 okhttp3 作为 reporter -->
        <dependency>
            <groupId>io.zipkin.reporter2</groupId>
            <artifactId>zipkin-sender-okhttp3</artifactId>
        </dependency>
    </dependencies>
```

### 需要链路的模块添加dubbo-zipkin-provider.xml

localServiceName需要改成当前模块的名字比用用户模块可以改成user(自定义为了分清模块)

```java
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!-- 2. zipkin 相关的配置 -->
    <!-- 使用 OKHttp 来发送 trace 信息到 Zipkin Server。这里的 Zipkin Server 启动在本地 -->
    <bean id="sender" class="zipkin2.reporter.beans.OkHttpSenderFactoryBean">
        <property name="endpoint" value="http://118.184.218.184:9411/api/v2/spans"/>
    </bean>

    <bean id="tracing" class="brave.spring.beans.TracingFactoryBean">
        <property name="localServiceName" value="user-provider2"/>
        <property name="spanReporter">
            <bean class="zipkin2.reporter.beans.AsyncReporterFactoryBean">
                <property name="sender" ref="sender"/>
                <!-- wait up to half a second for any in-flight spans on close -->
                <property name="closeTimeout" value="500"/>
            </bean>
        </property>
        <property name="currentTraceContext">
            <bean class="brave.spring.beans.CurrentTraceContextFactoryBean">
                <property name="scopeDecorators">
                    <bean class="brave.context.slf4j.MDCScopeDecorator" factory-method="create"/>
                </property>
            </bean>
        </property>
    </bean>

</beans>
```

#### 设置过滤器

```java
提供者需要添加
dubbo.provider.filter=tracing
消费者需要添加
dubbo.consumer.filter=tracing
既是提供者又是消费者添加
dubbo.provider.filter=tracing
dubbo.consumer.filter=tracing
```

#### 设置取消消费检查服务以来

```java
dubbo.consumer.check=false
```

#### springboot引用xml的配置

```java
@ImportResource(value = "/dubbo-zipkin-provider2.xml")
```

![](/images/1.png)

![](/images/2.png)

demo：https://github.com/ciweigg2/springboot-dubbo-nacos-zipkin