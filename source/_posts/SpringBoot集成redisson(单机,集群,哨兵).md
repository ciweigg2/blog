title: SpringBoot集成自己实现redisson-spring-boot-starter(单机,集群,哨兵,主从,云托管)
date: 2019-07-26 17:52:04
tags: [springboot,redis,redisson]
categories: [综合]
---
### redisson-spring-boot-starter

参照一些大神例子自己实现的starter呀

目前有很多项目还在使用jedis的 `setNx` 充当分布式锁,然而这个锁是有问题的,redisson是java支持redis的redlock的`唯一`实现,
集成该项目后只需要极少的配置.就能够使用redisson的全部功能. 目前支持
`集群模式`,`云托管模式`,`单Redis节点模式`,`哨兵模式`,`主从模式` 配置

<!--more-->

### 如何存储数据?(目前实现了三个对象模板)

1.RedissonObject 这个是比较通用的模板,任何对象都可以存在这里面,在spring 容器中注入对象即可 [demo实例](https://github.com/ciweigg/redisson-spring-boot-starter/tree/master/readme/object.md)

```java
@Autowired
private RedissonObject redissonObject;
```

2.RedissonBinary 这个是存储二进制的模板.可以存放图片之内的二进制文件,在spring 容器中注入对象即可 [demo实例](https://github.com/ciweigg/redisson-spring-boot-starter/tree/master/readme/binary.md)

```java
@Autowired
private RedissonBinary redissonBinary;
```

3.RedissonCollection 这个是集合模板,可以存放`Map`,`List`,`Set`集合元素,在spring 容器中注入对象即可 [demo实例](https://github.com/ciweigg/redisson-spring-boot-starter/tree/master/readme/collection.md)

```java
@Autowired
private RedissonCollection redissonCollection;
```

### 怎么使用呢

`添加maven`

``` 
<dependency>
    <groupId>com.github.ciweigg</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>${laster.version}/version>
</dependency>
```

properties的配置：[properties](https://github.com/ciweigg/redisson-spring-boot-starter/tree/master/readme/properties.md)

yml的配置：[yml](https://github.com/ciweigg/redisson-spring-boot-starter/tree/master/readme/yml.md)

### 更多参数配置

请参考：[参数配置](https://github.com/ciweigg/redisson-spring-boot-starter/tree/master/readme/att)

这是我自己封装的redisson-springboot-starter源码很简单可以参考：https://github.com/ciweigg/redisson-spring-boot-starter

对啦还有个实现呀 可以直接注入redisTemplate 使用了redisson的连接工厂实现集群的

```
@Autowired
private RedisTemplate redisTemplate;
```

参考：https://gitee.com/ztp/redisson-spring-boot-starter