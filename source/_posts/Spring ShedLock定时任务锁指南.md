cover: http://ciwei2.cn-sh2.ufileos.com/103.jpg
title: Spring ShedLock分布式定时任务锁指南
date: 2018-12-24 11:17:38
tags: [定时任务,ShedLock]
categories: [综合]
---
### 1 概述
Spring为定时任务提供了一个易于实现的API。在没有部署应用程序的多个实例之前，它很有效。默认情况下，Spring无法处理多个实例上的调度程序同步，而是在每个节点上同时执行作业。

在本篇教程中，我们将了解ShedLock - Java库，它确保我们的计划任务只能同时运行一次，并且可以代替Quartz。

demo地址：https://github.com/ciweigg2/springboot-shedlock/tree/master

shedlock可以在集群环境下只有一个name为TaskScheduler_scheduledTask的实例运行

运行2个springboot-shedlock项目 @SchedulerLock(name = "TaskScheduler_scheduledTask")name为相同的情况下会等锁释放后抢夺资源去执行，name不同的情况下各自执行任务，name相同的情况下满足了简单的集群环境

<!--more-->

### 2 Maven依赖
为了使用Spring ShedLock，我们需要添加shedlock-spring依赖项：

```java
<dependency>
    <groupId>net.javacrumbs.shedlock</groupId>
    <artifactId>shedlock-spring</artifactId>
    <version>2.2.0</version>
</dependency>
```

### 3 配置
注意，ShedLock仅适用于具有共享数据库的环境。它在数据库中创建一个表或文档，用于存储有关当前锁的信息。

目前，ShedLock支持Mongo，Redis，Hazelcast，ZooKeeper以及任何带有JDBC驱动程序的东西。

如下示例，我们将使用PostgreSQL数据库。为了使它工作，我们需要提供ShedLock的JDBC依赖：

```java
<dependency>
    <groupId>net.javacrumbs.shedlock</groupId>
    <artifactId>shedlock-provider-jdbc-template</artifactId>
    <version>2.2.0</version>
</dependency>
```

接下来，我们需要为ShedLock创建一个数据库表，以保留有关调度程序锁的信息：

```java
CREATE TABLE shedlock(
  name VARCHAR(64),
  lock_until TIMESTAMP(3) NULL,
  locked_at TIMESTAMP(3) NULL,
  locked_by  VARCHAR(255),
  PRIMARY KEY (name)
)
```
还需要提供的另一个配置是Spring配置类上的@EnableScheduling和@EnableSchedulerLock注释：

```java
@Configuration
@EnableScheduling
@EnableSchedulerLock(defaultLockAtMostFor = "PT30S")
public class ShedlockConfig {

    @Bean
    public LockProvider lockProvider(DataSource dataSource) {
        return new JdbcTemplateLockProvider(dataSource);
    }

    @Bean
    public ScheduledLockConfiguration scheduledLockConfiguration(LockProvider lockProvider) {
        return ScheduledLockConfigurationBuilder
                .withLockProvider(lockProvider)
                .withPoolSize(10)
                .withDefaultLockAtMostFor(Duration.ofMinutes(10))
                .build();
    }

}
```
defaultLockAtMostFor参数指定在执行节点结束时应保留锁的默认时间量。它使用ISO8601 Duration格式。

在下一节中，我们将了解如何重写此默认值。

### 4 创建任务
要创建由ShedLock处理的计划任务，我们只需将方法上配置@Scheduled和@SchedulerLock注释：

```java
@Component
public class Task2Scheduler {

    @Scheduled(cron = "*/15 * * * * *")
    @SchedulerLock(name = "TaskScheduler_scheduledTask", 
      lockAtLeastForString = "PT5M", lockAtMostForString = "PT14M")
    public void scheduledTask() {
        log.info("开始睡眠当前时间"+System.currentTimeMillis());
//        Thread.sleep(1000 * 60 *3);
        log.info("结束睡眠当前时间"+System.currentTimeMillis());
    }
}
```

首先，让我们来看看@Scheduled。它支持cron格式，这个表达式意味着“每15分钟”。

接下来，看看@SchedulerLock，name参数必须是唯一的，ClassName_methodName通常足以实现它。我们不希望同时有多个相同名称方法运行，所以ShedLock使用唯一名称来实现该目的。

我们还添加了几个可选参数。

首先，我们添加了lockAtLeastForString，以便我们可以在方法调用之间产生时间间隔。使用“PT5M”意味着此方法至少要锁定5分钟。换句话说，这意味着这种方法可以由ShedLock运行，而不是每五分钟运行一次。

接下来，我们添加了lockAtMostForString来指定在执行节点完成时应该保留多长时间。使用“PT14M”意味着它将被锁定不超过14分钟。

在正常情况下，ShedLock会在任务完成后直接释放锁(lockAtMostFor属性，指定执行节点死亡时应该保留锁的时间。这只是后备，正常情况下，锁定会在任务完成后立即释放。)。但是，实际上，我们没有必要这样做，因为@EnableSchedulerLock中提供了默认值，但我们选择在此处重写它。

### 5 配置mysql数据库
```java
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.45</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        
配置application.properties
# mysql
spring.datasource.url=jdbc:mysql://localhost:3306/shedlock?useUnicode=true&characterEncoding=UTF-8&useSSL=false
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.max-idle=10
spring.datasource.max-wait=10000
spring.datasource.min-idle=5
spring.datasource.initial-size=5
```
### 6 总结
在本文中，我们学习了如何使用ShedLock创建和同步计划任务。