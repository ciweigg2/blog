---
title: SpringBoot整合Flyway完成数据库持久化迭代更新
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
  - flyway
categories:
  - 综合
date: 2020-01-05 19:30:54
---

每次服务的代码更新部署，难免会存在`数据库结构`的变更以及`字典数据的添加`，手动`执行更新脚本`是一个`耗时耗力`的工作，而且还会出现遗漏或者其他状况，`SpringBoot`内部集成了一个自动执行数据库脚本的第三方依赖`Flyway`来解决这个繁琐的问题

<!--more-->

## 什么是Flyway

官网给出的定义是`Version control for your database. Robust schema evolution across all your environments. With ease, pleasure and plain SQL.`（数据库的版本控制，在所有环境中进行稳健的架构演变，轻松，愉快和简单的SQL。）

`Flyway` 是一款开源的数据库版本管理工具，它更倾向于规约优于配置的方式。

`Flyway` 可以独立于应用实现管理并跟踪数据库变更，支持数据库版本自动升级，并且有一套默认的规约，不需要复杂的配置，`Migrations` 可以写成 `SQL 脚本`，也可以写在 Java 代码中，不仅支持 `Command Line` 和 `Java API`，还支持 Build 构建工具和 `Spring Boot` 等，同时在分布式环境下能够安全可靠地`升级数据库`，同时也`支持失败恢复`等。

## Flyway运行原理

当我们运行配置使用`Flyway`的应用程序时，会自动在配置数据源的数据库内创建一个名为
**flyway_schema_history**的表，该表内存放了`数据库的历史记录`信息。
![image.png](/images/2020/01/05/be44db20-2fae-11ea-a9cb-bb9279b30331.png)
然后通过扫码应用程序的`/reosurces/db/migration`目录下的历史版本脚本SQL文件，文件格式为：`V?__desc.sql`，如：`V1__init-db.sql`，**根据版本号进行排序后，获取最大的版本号与`flyway_schema_history`表内执行成功的最大版本号进行比对，如果项目内版本较高，则自动执行脚本文件。**
![image.png](/images/2020/01/05/c1d072e0-2fae-11ea-a9cb-bb9279b30331.png)

## 创建项目

通过`idea`工具创建`SpringBoot`项目，在`pom.xml`添加相关依赖如下所示：

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.flywaydb</groupId>
        <artifactId>flyway-core</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

### 添加数据库配置

在`application.yml`配置文件内添加数据源信息，如下所示：

```java
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/flyway
    username: root
    password: 123456
    type: com.zaxxer.hikari.HikariDataSource
```

### 添加Flyway版本脚本

![image.png](/images/2020/01/05/cb8ecd90-2fae-11ea-a9cb-bb9279b30331.png)

> 脚本比较简单，大家可以任意添加一些SQL来查看结构或者数据变动。

`db.migration`目录是`SpringBoot`在整合`Flyway`时默认读取版本脚本的目录，我们可以在`application.yml`配置`spring.flyway.locations`参数进行修改。

## 测试

当我们**启动项目时**，会自动比对脚本的版本，在`db.migration`目录内找到`V1.1__add_logging.sql`为最高版本，拿着`1.1`再去`flyway_schema_history`表内执行成功最大的版本比对，如果低于`1.1`则自动执行`V1.1_add_logging.sql`脚本内容，否则跳过。

### flyway_schema_history表

每次启动项目如果存在可更新的脚本信息，执行完成后会自动在`flyway_schema_history`表内添加一条记录。

| installed_rank | version | description | type | script               |
| :------------- | :------ | :---------- | :--- | :------------------- |
| 1              | 1       | init        | SQL  | V1__init.sql         | 
| 2              | 1.1     | add logging | SQL  | V1.1_add_logging.sql |

## 敲黑板，划重点

本章简单的介绍了`Flyway`的基本使用方法，它很强大，功能远远不止于此，使用脚本统一自动执行可大大减少手动执行出现的遗漏、错误等
存在既有道理，为什么不尝试使用呢