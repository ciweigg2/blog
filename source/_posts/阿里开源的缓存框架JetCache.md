cover: http://ciwei2.cn-sh2.ufileos.com/8.jpg
title: 阿里开源的缓存框架JetCache
date: 2019-04-15 10:42:19
tags: [jetCache]
categories: [综合]
---
之前一直在用Spring Cache进行接口数据的缓存，主要是Spring Cache在对具体key缓存失效时间的设置不是很方法，还要自己去扩展，无意中发现了阿里的JetCache。大部分的需求都能满足，并且有一些很实用的功能，今天给大家介绍下。

JetCache是一个基于Java的缓存系统封装，提供统一的API和注解来简化缓存的使用。 JetCache提供了比SpringCache更加强大的注解，可以原生的支持TTL、两级缓存、分布式自动刷新，还提供了Cache接口用于手工缓存操作。 当前有四个实现，RedisCache、TairCache（此部分未在github开源）、CaffeineCache(in memory)和一个简易的LinkedHashMapCache(in memory)，要添加新的实现也是非常简单的。

<!--more-->

GitHub：https://github.com/alibaba/jetcache

### 全部特性:

* 通过统一的API访问Cache系统
* 通过注解实现声明式的方法缓存，支持TTL和两级缓存
* 通过注解创建并配置Cache实例
* 针对所有Cache实例和方法缓存的自动统计
* Key的生成策略和Value的序列化策略是可以配置的
* 分布式缓存自动刷新，分布式锁 (2.2+)
* 异步Cache API (2.2+，使用Redis的lettuce客户端时)
* Spring Boot支持

### 体验一下

增加Maven配置：

```java
<dependency>
    <groupId>com.alicp.jetcache</groupId>
    <artifactId>jetcache-starter-redis</artifactId>
    <version>2.5.11</version>
</dependency>
```

配置内容：

```java
# 采用Java序列化存储
jetcache.remote.default.valueDecoder = java
# Key的转换器
jetcache.remote.default.keyConvertor = fastjson
jetcache.remote.default.type = redis
jetcache.remote.default.poolConfig.minIdle = 5
jetcache.remote.default.poolConfig.maxIdle = 20
jetcache.remote.default.poolConfig.maxTotal = 50
# 是否加入缓存key前缀
jetcache.areaInCacheName = false
jetcache.remote.default.valueEncoder = java
# 缓存类型。tair、redis为当前支持的远程缓存；linkedhashmap、caffeine为当前支持的本地缓存类型
jetcache.local.default.type = linkedhashmap
# 控制台输出统计数据，统计间隔，0表示不统计
jetcache.statIntervalMinutes = 15
jetcache.local.default.keyConvertor = fastjson
jetcache.remote.default.host = 127.0.0.1
jetcache.remote.default.port = 6379
```

* remote 表示远程缓存
* local表示本地缓存

### 启动类开启缓存：

```java
@SpringBootApplication
@EnableMethodCache(basePackages = "com.cxytiandi.jetcache")
@EnableCreateCacheAnnotation
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class);
    }
}

@EnableMethodCache
用于激活@Cached注解的使用
* @EnableCreateCacheAnnotation
用于激活@CreateCache注解的使用
```

定义一个简单的实体类来作为数据的缓存，必须实现Serializable接口。

```java
@Data
public class User implements Serializable {

    private Long userId;

    private String userName;

}
```

```java
@CreateCache使用

@CreateCache(expire = 100)
private Cache<Long, User> userCache;

User user = new User();
user.setId(1L);
user.setName("yinjihuan");
// 新增缓存
userCache.put(1L, user);

// 删除缓存
userCache.remove(1L);
```

用起来很简单，就像操作本地Map一样，@CreateCache中有很多配置需要我们自己去指定，不指定则使用默认的，关于配置请查看文档：https://github.com/alibaba/jetcache/wiki/CreateCache_CN

```java
@Cached使用

@Cached(name="getUser.", key="#id", expire = 8, cacheType=CacheType.BOTH)
@Override
public User getUser(Long id) {
    User user = new User();
    user.setId(1L);
    user.setName("yinjihuan");
    return user;
}

name
缓存名称
key
缓存key,追加到name后面构成唯一的缓存key, 使用SpEL指定key，如果没有指定会根据所有参数自动生成。
expire
缓存失效时间
cacheType
缓存的类型，包括CacheType.REMOTE、CacheType.LOCAL、CacheType.BOTH。如果定义为BOTH，会使用LOCAL和REMOTE组合成两级缓存
```

更多配置的介绍请查看文档：https://github.com/alibaba/jetcache/wiki/MethodCache_CN

测试例子：https://github.com/ciweigg2/springboot-jetcache