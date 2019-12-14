title: Spring Boot集成Sharding-JDBC实现读写分离
date: 2019-07-15 21:35:02
tags: [springboot,shardingsphere,shardingjdbc]
categories: [综合]
---
### Sharding-JDBC简介

Sharding-JDBC是的分布式数据库中间件解决方案。Sharding-JDBC、Sharding-Proxy和Sharding-Sidecar（计划中）是3款相互独立的产品，共同
组成了ShardingSphere。Sharding-JDBC定位于轻量级的Java框架，它使用客户端直连数据库，可理解为增强版的JDBC驱动，完全兼容JDBC和各种ORM框架

<!--more-->

为了缓解单表数据千万的问题

* 适用于任何基于Java的ORM框架，如：JPA, Hibernate, Mybatis, Spring JDBC Template或直接使用JDBC。
* 基于任何第三方的数据库连接池，如：DBCP, C3P0, BoneCP, Druid, HikariCP等。
* 支持任意实现JDBC规范的数据库。目前支持MySQL，Oracle，SQLServer和PostgreSQL。

架构图如下：

![](/images/20190715213910.png)

支持以下的功能：

* 分库分表
* 读写分离
* 柔性事务
* 分布式主键
* 分布式治理能力

### 工程准备

首先需要mysql主从复制 参考docker-compose安装mysql一主多从

在主库里面执行以下的数据库初始化脚本：

```
CREATE DATABASE /*!32312 IF NOT EXISTS*/`cool` /*!40100 DEFAULT CHARACTER SET utf8 */;

USE `cool`;

DROP TABLE IF EXISTS `user`;

CREATE TABLE `user` (
  `id` bigint(64) NOT NULL AUTO_INCREMENT,
  `username` varchar(12) NOT NULL,
  `password` varchar(30) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx-username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=12 DEFAULT CHARSET=utf8;
```

主从数据库已经搭建好了，所以执行完上面的脚本后，2个从库应该也有user表。

### 案例讲解

在本篇文章中使用Spring Boot 2.0.3+MyBatis+Druid+Sharding-JDBC+MySQL进行读写分离的案件讲解。

在工程的pom文件引入以下的依赖，包括Spring Boot的Web起步依赖spring-boot-starter-web，mybatis的起步依赖mybatis-spring-boot-starter，
mysql的连机器，连接池druid的起步依赖druid-spring-boot-starter，sharding-jdbc的起步依赖sharding-jdbc-spring-boot-starter。代码如下：

```
  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>

        <dependency>
            <groupId>io.shardingsphere</groupId>
            <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
            <version>3.1.0.M1</version>
        </dependency>
```

在application.yml中添加：

```java
master-host: 127.0.0.1
slave1-host: 127.0.0.1
slave2-host: 127.0.0.1

sharding:
  jdbc:
    dataSource:
      names: db-test0,db-test1,db-test2
      # 配置主库
      db-test0:
        type: com.alibaba.druid.pool.DruidDataSource
        driverClassName: com.mysql.jdbc.Driver
        url: jdbc:mysql://${master-host}:3340/cool?useUnicode=true&characterEncoding=utf8&tinyInt1isBit=false&useSSL=false&serverTimezone=GMT
        username: root
        password: 
        #最大连接数
        maxPoolSize: 20
      db-test1:
        # 配置第一个从库
        type: com.alibaba.druid.pool.DruidDataSource
        driverClassName: com.mysql.jdbc.Driver
        url: jdbc:mysql://${slave1-host}:3341/cool?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true&useSSL=false&serverTimezone=GMT
        username: root
        password: 
        maxPoolSize: 20
      db-test2:
        # 配置第二个从库
        type: com.alibaba.druid.pool.DruidDataSource
        driverClassName: com.mysql.jdbc.Driver
        url: jdbc:mysql://${slave2-host}:3342/cool?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true&useSSL=false&serverTimezone=GMT
        username: root
        password: 
        maxPoolSize: 20
    config:
      masterslave:
        # 配置读写分离
        load-balance-algorithm-type: round_robin # 配置从库选择策略，提供轮询与随机，这里选择用轮询//random 随机 //round_robin 轮询
        name: db1s2
        master-data-source-name: db-test0
        slave-data-source-names: db-test1,db-test2
    props:
      sql:
        show: true # 开启SQL显示，默认值: false，注意：仅配置读写分离时不会打印日志！！！

server.port: 8085
mybatis.config-location: classpath:META-INF/mybatis-config.xml
```

sharding.jdbc.dataSource.names配置的是数据库的名称，就是多个数据源的名称。
sharding.jdbc.dataSource配置多个数据源。需要配置数据库名称，和上面配置的对应。以及数据的配置，包括连接池的类型、连接器、数据库地址、
数据库账户密码信息等。
sharding.jdbc.config.masterslave.load-balance-algorithm-type查询时的负载均衡算法，目前有2种算法，round_robin（轮询）和random（随机）。
sharding.jdbc.config.masterslave.master-data-source-name主数据源名称。
sharding.jdbc.config.masterslave.slave-data-source-names从数据源名称，多个用逗号隔开。

![](/images/20190715214601.png)

### 案例验证

写2个接口，代码如下：

```java
@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/users")
    public Object list() {
        return userService.list();
    }

    @GetMapping("/add")
    public Object add(@RequestParam Integer id,@RequestParam String username,@RequestParam String  password) {
        User user = new User();
        user.setId(id);
        user.setUsername(username);
        user.setPassword(password);
        return userService.addUser(user);
    }
}
```

已经开启了数据库的CRUD日志，日志目录在/var/lib/mysql目录下(因为docker安装的所以挂载到宿主机了呀)。

### 开启3个库的日志查询(不建议生产使用呀)：

查看日志目录，并开启sql语句的日志：

```
mysql>  show variables like '%general_log%';
mysql>  set global general_log=on;
```

开启后，重启Mysql ，上述开启日志配置将失效。

调用2个接口，可以在主库对应主机的日志目录下查看插入数据的日志：

```java
2019-07-15T14:24:00.788254Z	 36 Query	select @@session.transaction_read_only
2019-07-15T14:24:00.788254Z	 36 Query	INSERT INTO user (
          id, username, password
        )
        VALUES (
        134,
        'forezp134',
        '1233edwd'
        )
```

从库对应主机的日志目录下查看查询数据的日志：

```java
2019-07-15T14:24:00.788254Z	 36 Query	SELECT u.* FROM user u
```

测试一下发现读写分离成功了

demo：https://github.com/ciweigg2/sharding-jdbc-example/tree/master/sharding-jdbc-master-slave