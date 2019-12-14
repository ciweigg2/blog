cover: http://ciwei2.cn-sh2.ufileos.com/122.jpg
title: springboot集成java应用监控平台javamelody
date: 2019-04-25 14:52:04
tags: [javamelody]
categories: [综合]
---
文档详情地址：https://github.com/javamelody/javamelody/wiki/SpringBootStarter

<!--more-->

```java
        <dependency>
            <groupId>net.bull.javamelody</groupId>
            <artifactId>javamelody-spring-boot-starter</artifactId>
            <version>1.77.0</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>com.lowagie</groupId>
            <artifactId>itext</artifactId>
            <version>2.1.7</version>
            <exclusions>
                <exclusion>
                    <artifactId>bcmail-jdk14</artifactId>
                    <groupId>bouncycastle</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>bcprov-jdk14</artifactId>
                    <groupId>bouncycastle</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>bctsp-jdk14</artifactId>
                    <groupId>bouncycastle</groupId>
                </exclusion>
            </exclusions>
        </dependency>
```

访问地址：http://localhost:8081/actuator/monitoring

去除spring-boot-starter-actuator依赖

访问地址：http://localhost:8080/monitoring

如果想监控项目中的某些类或方法，可以使用注解 @MonitoredWithSpring ，将此注解添加到类或方法上就行了呀，