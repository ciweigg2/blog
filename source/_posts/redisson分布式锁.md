title: redisson分布式锁
date: 2019-07-18 22:29:47
tags: [redisson]
categories: [综合]
---
### 快速入门

介绍

lock-spring-boot-starter是一个基于Spring Boot的starter机制编写的分布式锁工具。 与其他分布式锁不同的是,使用起来更方便快捷,只需要通过注解@Lock的方式即可实现对方法进行加锁。

<!--more-->

### 锁类型支持

可重入锁,公平锁,联锁,红锁,读锁,写锁

### 入门示例

1. 创建Spring Boot项目

2. 引入maven依赖

```
<dependency>
     <groupid>io.gitee.tooleek</groupid>
     <artifactid>lock-spring-boot-starter</artifactid>
     <version>1.1.0</version>
</dependency>
```

3. 在Spring Boot的项目配置文件application.yml中添加相应的配置，如：

```
lock-config: 
     pattern: single #redis模式配置，single：单机模式，cluster:集群模式，replicated:云托管模式,sentinel:哨兵模式，master_slave：主从模式
     # 不同的redis模式对应不同的配置方式，single-server对应的就是单机模式，具体参数意义可参考redisson的配置参数说明
     single-server: 
         address: 127.0.0.1
         port: 6379
         password: 123456
```

4. 在需要使用分布式锁的方法上面使用@Lock注解，锁的关键字使用@Key，如:

```
@Lock
 public void hello(String ces, @Key String orderNo) {
 	System.out.println("hello");
 }
```

注：如果需要配置不同类型的锁，可以直接变更@Lock的参数值即可，默认是可重入锁

@Lock提供四个参数可以配置： lockType:锁类型 leaseTime:加锁时间 waitTime:最长等待时间 timeUnit:锁时长单位

DEMO地址如下：https://github.com/ciweigg2/lock-spring-boot-starter-demo

### 使用说明

#### @Lock注解

@Lock注解用于标识需要在某个方法上使用分布式锁,@Lock提供三个参数用于设置锁的相关参数,参数如下：

lockType：锁类型设置,支持6种常用锁,值类型为：LockType,可选值为：LockType.REENTRANT(可重入锁)、LockType.FAIR(公平锁)、LockType.MULTI(联锁)、LockType.RED(红锁)、LockType.READ(读锁)、LockType.WRITE(写锁)

waitTime：等待锁的时间,超过该时长依然无法获取到锁则不会执行程序

leaseTime：加锁时长,超过该时长自动解锁,所以设置该值时一定要设置一个合理的值

timeUnit：锁时长单位,值类型为：TimeUnit,默认值为TimeUnit.SECONDS

#### @Key注解

@Key注解用于标识加锁的参数,红锁和联锁支持使用多个@Key,@Key提供三种注解方式

方法参数注解：在方法的参数前面使用@Key标识用来作为加锁条件的参数

```
@Lock(lockType=LockType.REENTRANT,waitTime=20,leaseTime=10)
public void lock(String ces, @Key String orderNo) {
	System.out.println("hello");
}
```
					
方法体注解：在方法体上面使用@Key标识用来作为加锁条件的参数

```
@Lock(lockType=LockType.REENTRANT,waitTime=20,leaseTime=10)
@Key({"DemoEntity.orderNo"})
public void lock(DemoEntity entity) {
	System.out.println("hello");
}
```
					
参数类属性注解：在方法的参数类属性上面使用@Key标识用来作为加锁条件的参数

```
@Setter
@Getter
public class PropertiesLockEntity {
    @Key
    private String id;
    private Integer count;
}
```

测试一下呀：

```
@Lock(lockType=LockType.REENTRANT,waitTime=20,leaseTime=10)
public void lock(PropertiesLockEntity demoEntity) {
	System.out.println("hello");
}
```

注：如果在方法上使用了@Lock但并未进行指定Key那么会默认锁整个方法

### 配置参数lock-config

用于设置Redis相关的连接信息和配置

pattern：Redis模式配置,single：单机模式，cluster:集群模式，replicated:云托管模式,sentinel:哨兵模式，master_slave：主从模式

不同的redis模式对应不同的配置方式，single-server对应的就是单机模式，具体参数意义可参考redisson的配置参数说明

single-server：单机模式参数配置

cluster-server：集群模式参数配置

master-slave-server：主从模式参数配置

sentinel-server：哨兵模式参数配置

replicated-server：云托管模式参数配置

如果需要修改redis的key前缀，源码中LockCommonConstant的KEY_PREFIX修改才行呀