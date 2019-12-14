cover: http://ciwei2.cn-sh2.ufileos.com/7.jpg
title: 跟我学Nacos之SpringBoot-Dubbo整合Nacos注册中心-01
date: 2019-01-13 12:58:38
tags: [nacos,dubbo]
categories: [nacos]
---
## 搭建springboot-dubbo项目集成nacos注册中心

阿里小马哥demo地址:https://github.com/apache/incubator-dubbo-spring-boot-project

<!--more-->

### 项目结构

![](/images/20190113131620.png)

### 最外层POM添加依赖

```java
  <properties>
        <java.version>1.8</java.version>
        <java.source.version>1.8</java.source.version>
        <java.target.version>1.8</java.target.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.1.2.RELEASE</spring-boot.version>
        <dubbo.version>2.6.5</dubbo.version>
        <dubbo-registry-nacos.version>0.0.2</dubbo-registry-nacos.version>
        <nacos-client.version>0.6.2</nacos-client.version>
        <!-- Build args -->
        <argline>-server -Xms256m -Xmx512m -XX:PermSize=64m -XX:MaxPermSize=128m -Dfile.encoding=UTF-8
            -Djava.net.preferIPv4Stack=true
        </argline>
        <arguments/>

        <!-- Maven plugins -->
        <maven-jar-plugin.version>3.0.2</maven-jar-plugin.version>
        <maven-compiler-plugin.version>3.6.0</maven-compiler-plugin.version>
        <maven-source-plugin.version>3.0.1</maven-source-plugin.version>
        <maven-jacoco-plugin.version>0.8.1</maven-jacoco-plugin.version>
        <maven-gpg-plugin.version>1.5</maven-gpg-plugin.version>
        <apache-rat-plugin.version>0.12</apache-rat-plugin.version>
        <maven-release-plugin.version>2.5.3</maven-release-plugin.version>
        <maven-surefire-plugin.version>2.19.1</maven-surefire-plugin.version>
        <alibaba-spring-context-support.version>1.0.2</alibaba-spring-context-support.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>dubbo-api</artifactId>
                <version>0.0.1-SNAPSHOT</version>
            </dependency>

            <!-- Dubbo dependencies -->
            <dependencalibaba</groupId>
                <artifactId>dubbo-dependencies  <artifactId>dubbo-dependencies-bom</artifactId>
                <version>${dubbo.version}</version>
            </dependency>

            <!-- Dubbo -->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>dubbo</artifactId>
                <version>${dubbo.version}</version>
                <exclusions>
                    <exclusion>
                        <groupId>org.springframework</groupId>
                        <artifactId>spring</artifactId>
                    </exclusion>
                    <exclusion>
                        <groupId>javax.servlet</groupId>
                        <artifactId>servlet-api</artifactId>
                    </exclusion>
                    <exclu                       <groupId>log4j</groupId>
                               <artifactId>log4j</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>

            <!-- Dubbo Nacos registry dependency -->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>dubbo-registry-nacos</artifactId>
                <version>${dubbo-registry-nacos.version}</version>
            </dependency>

            <!-- Keep latest Nacos client version -->
            <dependency>
                <groupId>com.alibaba.nacos</groupId>
                <artifactId>nacos-client</artifactId>
                <version>${nacos-client.version}</version>
            </dependency>

            <dependency>
                <groupId>com.alibaba.boot</groupId>
                <artifactId>dubbo-spring-boot-starter</artifactId>
                <version>0.2.1-SNAPSHOT</version>
            </dependency>

        </dependencies>
    </dependencyManagement>
```

### 新建 api provider consumer模块

### 在api层新建api

```java
public interface DemoService {

    String sayHello(String name);

}
```

### consumer新增springboot启动类和controller层
```java
@EnableAutoConfiguration
@SpringBootApplication
@NacosPropertySource(
        name = "custom",
        dataId = "nacos_config",
        first = true,
        groupId = "nacos_group",
        autoRefreshed = true,
        before = SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME,
        after = SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME
)
public class DubboRegistryNacosConsumerBootstrap {

    public static void main(String[] args) {
        SpringApplication.run(DubboRegistryNacosConsumerBootstrap.class ,args);
    }
}
```

* controller

```java
@RestController
public class DubboController {

    @Reference(version = "${demo.service.version}")
    private DemoService demoService;

    @RequestMapping(value = "/sayHello")
    public String dubboSayHello(){
        return demoService.sayHello("sayHello");
    }
```

* application.yml

```java
spring:
  application:
    name: dubbo-registry-nacos-consumer-sample

demo:
  service:
    version: 1.0.0

dubbo:
  registry:
    address: nacos://118.184.218.184:8848
  # 消费者默认不检测是否有服务注册
  consumer:
    check: false
```

* maven

```java
    <dependencies>

        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
        </dependency>

        <!-- Dubbo -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
        </dependency>

        <!-- Netty -->
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
        </dependency>

        <!-- Dubbo Nacos registry dependency -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo-registry-nacos</artifactId>
        </dependency>

        <!-- Keep latest Nacos client version -->
        <dependency>
            <groupId>com.alibaba.nacos</groupId>
            <artifactId>nacos-client</artifactId>
        </dependency>

        <dependency>
            <groupId>com.example</groupId>
            <artifactId>dubbo-api</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
```

### provider模块配置

```java
@EnableAutoConfiguration
public class DubboRegistryNacosProviderBootstrap {

    public static void main(String[] args) {
        new SpringApplicationBuilder(DubboRegistryNacosProviderBootstrap.class)
                .web(WebApplicationType.NONE)
                .run();
    }
}
```

```java
@Service(version = "${demo.service.version}")
public class DemoServiceImpl implements DemoService {

    /**
     * The default value of ${dubbo.application.name} is ${spring.application.name}
     */
    @Value("${dubbo.application.name}")
    private String serviceName;

    public String sayHello(String name) {
        RpcContext rpcContext = RpcContext.getContext();
        System.out.println(String.format("Service [name :%s , port : %d] %s(\"%s\") : Hello,%s",
                serviceName,
                rpcContext.getLocalPort(),
                rpcContext.getMethodName(),
                name,
                name));
        return String.format("[%s] : Hello, %s", serviceName, name);
    }
}
```

* application.properties

```java
# Spring boot application
spring.application.name=dubbo-registry-nacos-provider-sample

# Base packages to scan Dubbo Component: @com.alibaba.dubbo.config.annotation.Service
dubbo.scan.base-packages=com.mxc.service.impl

# Dubbo Application
## The default value of dubbo.application.name is ${spring.application.name}
## dubbo.application.name=${spring.application.name}

# Dubbo Protocol
dubbo.protocol.name=dubbo
## Random port
dubbo.protocol.port=-1

## Dubbo Registry
dubbo.registry.address=nacos://118.184.218.184:8848

## DemoService version
demo.service.version=1.0.0
```

* maven依赖

```java
    <dependencies>

        <!-- Spring Boot dependencies -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
        </dependency>

        <!-- Dubbo -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
        </dependency>

        <!-- Netty -->
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
        </dependency>

        <!-- Dubbo Nacos registry dependency -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo-registry-nacos</artifactId>
        </dependency>

        <!-- Keep latest Nacos client version -->
        <dependency>
            <groupId>com.alibaba.nacos</groupId>
            <artifactId>nacos-client</artifactId>
        </dependency>

        <dependency>
            <groupId>com.example</groupId>
            <artifactId>dubbo-api</artifactId>
        </dependency>

        <!-- Dubbo -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.5</version>
        </dependency>

    </dependencies>
```

### 启动测试 依次启动provider和consumer

* 访问接口：http://localhost:8080/sayHello

### 配置中心

![](/images/20190113133056.png)

![](/images/20190113133115.png)

### 如果消费者中也提供了服务怎么办呢

* consumer层添加application.yml配置 只需要扫描实现类就行了 接口实现配置和提供者一样配置就ok了

  # 提供者需要配置扫描接口
dubbo:  
  scan:
    base-packages: com.mxc.web.impl
  protocol:
    port: -1

demo：https://github.com/ciweigg2/springboot-dubbo-nacos.git