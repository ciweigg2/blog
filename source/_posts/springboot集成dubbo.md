cover: http://ciwei2.cn-sh2.ufileos.com/120.jpg
title: springboot集成dubbo
date: 2018-10-07 16:02:38
tags: [springboot,dubbo]
categories: [dubbo]
top: true
---
### dubbo介绍

> 今年年初时，阿里巴巴开源的高性能服务框架dubbo又开始了新一轮的更新，还加入了Apache孵化器。原先项目使用了spring cloud之后，已经比较少用dubbo。目前又抽调回原来的行业应用部门，可能还会使用dubbo进行服务调用。趁着编写教材的机会来进行学习下。而且目前Dubbo也出了springboot的starter项目了，借着SpringBoot的东风，集成起来很方便，基本上就一个依赖包引入的问题了。废话不多说，开始吧~

<!--more-->

### 一点知识

对于没有接触过Dubbo的同学，可以先了解下相关知识。

### Dubbo简介

Dubbo 是阿里巴巴公司一个开源的高性能服务框架，致力于提供高性能和透明化的 RPC 远程服务调用方案，以及 SOA 服务治理方案，使得应用可通过高性能 RPC 实现服务的输出、输入功能和 Spring 框架无缝集成。Dubbo 包含远程通讯、集群容错和自动发现三个核心部分。

它提供透明化的远程方法调用，实现像调用本地方法一样调用远程方法，只需简单配置，没有任何 API 侵入。同时它具备软负载均衡及容错机制，可在内网替代 F5 等硬件负载均衡器，降低成本，减少单点。它可以实现服务自动注册与发现，不再需要写死服务提供方地址，注册中心基于接口名查询服务提供者的 IP 地址，并且能够平滑添加或删除服务提供者。

2011 年末，阿里巴巴在 GitHub 上开源了基于 Java 的分布式服务治理框架 Dubbo，之后它成为了国内该类开源项目的佼佼者，许多开发者对其表示青睐。同时，先后有不少公司在实践中基于 Dubbo 进行分布式系统架构。目前在 GitHub 上，它的 fork、star 数均已破万。

### Dubbo核心功能:

* 远程通讯，提供对多种基于长连接的 NIO 框架抽象封装，包括多种线程模型，序列化，以及“请求-响应”模式的信息交换方式。
* 集群容错，提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载均衡，失败容错，地址路由，动态配置等集群支持。
* 自动发现，基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地址透明，使服务提供方可以平滑增加或减少机器。

### Dubbo架构

1. 服务提供者 - 启动时在指定端口上暴露服务，并将服务地址和端口注册到注册中心上
2. 服务消费者 - 启动时向注册中心订阅自己感兴趣的服务，以便获得服务提供方的地址列表
3. 注册中心 - 负责服务的注册和发现，负责保存服务提供方上报的地址信息，并向服务消费方推送
4. 监控中心 - 负责收集服务提供方和消费方的运行状态，比如服务调用次数、延迟等，用于监控
5. 运行容器 - 负责服务提供方的初始化、加载以及运行的生命周期管理

![](/images/dubbo.jpg)

### 部署阶段

* 服务提供者在指定端口暴露服务，并向注册中心注册服务信息。
* 服务消费者向注册中心发起服务地址列表的订阅。


### 运行阶段

* 注册中心向服务消费者推送地址列表信息。
* 服务消费者收到地址列表后，从其中选取一个向目标服务发起调用。
* 调用过程服务消费者和服务提供者的运行状态上报给监控中心。


### 调用关系说明

1. 服务容器负责启动，加载，运行服务提供者。
2. 服务提供者在启动时，向注册中心注册自己提供的服务。
3. 服务消费者在启动时，向注册中心订阅自己所需的服务。
4. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
5. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
6. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

### Dubbo特点

> Dubbo 架构具有以下几个特点，分别是连通性、健壮性、伸缩性、以及向未来架构的升级性。

### 连通性
* 注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小
* 监控中心负责统计各服务调用次数，调用时间等，统计先在内存汇总后每分钟一次发送到监控中心服务器，并以报表展示
* 服务提供者向注册中心注册其提供的服务，并汇报调用时间到监控中心，此时间不包含网络开销
* 服务消费者向注册中心获取服务提供者地址列表，并根据负载算法直接调用提供者，同时汇报调用时间到监控中心，此时间包含网络开销
* 注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外
* 注册中心通过长连接感知服务提供者的存在，服务提供者宕机，注册中心将立即推送事件通知消费者
* 注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表
* 注册中心和监控中心都是可选的，服务消费者可以直连服务提供者

### 健壮性
* 监控中心宕掉不影响使用，只是丢失部分采样数据
* 数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
* 注册中心对等集群，任意一台宕掉后，将自动切换到另一台
* 注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯
* 服务提供者无状态，任意一台宕掉后，不影响使用
* 服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

### 伸缩性
* 注册中心为对等集群，可动态增加机器部署实例，所有客户端将自动发现新的注册中心
* 服务提供者无状态，可动态增加机器部署实例，注册中心将推送新的服务提供者信息给消费者

### 升级性
* 当服务集群规模进一步扩大，带动IT治理结构进一步升级，需要实现动态部署，进行流动计算，现有分布式服务架构不会带来阻力。下图是未来可能的一种架构：

![](/images/20181007160922.png)

### 节点角色说明

* Deployer	自动部署服务的本地代理
* Repository	仓库用于存储服务应用发布包
* Scheduler	调度中心基于访问压力自动增减服务提供者
* Admin	统一管理控制台
* Registry	服务注册与发现的注册中心
* Monitor	统计服务的调用次数和调用时间的监控中心

大家可访问官网文档：http://dubbo.apache.org/zh-cn/docs/user/quick-start.html，里面有详细说明和使用说明的。这里就不再阐述了。

### Dubbo集成和使用

> 基于官方的incubator-dubbo-spring-boot-project项目，在SpringBoot中集成起来很简单。

注意：由于本系列还是使用1.5.x版本进行讲解，所以使用的版本为0.1.x。若使用SpringBoot2.x的同学，可以使用0.2.x版本。

> 0.2.x springboot 2.x 长期维护
0.1.x springboot 1.x 暂时维护

### 这里为了方便，直接创建了一个接口工程，spring-boot-dubbo-api。

IHelloService.java
```java
/**
 * 定义一个接口
 * @author oKong
 *
 */
public interface IHelloService {
    
    String hello(String name);

}
```

### 服务提供者

创建一个spring-boot-dubbo-provider工程。引入pom依赖。

```java
        <!-- 引入api -->
        <dependency>
            <groupId>cn.lqdev.learning</groupId>
            <artifactId>spring-boot-dubbo-api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
        <!-- 引入dubbo依赖 -->
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>0.1.1</version>
        </dependency>
        <!-- 引入redis作为注册中心 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

注意：这里直接选用了redis作为注册中心使用。默认是zookeeper。

1.编写接口实现类。 HelloServiceImpl.java

```java
/**
 * 定义一个服务实现类
 * @author oKong
 *
 */
// 这里注意 此类@service是dubbo的
@Service(
        version = "${demo.service.version}", //版本
        application = "${dubbo.application.id}", //应用ID
        protocol = "${dubbo.protocol.id}", //协议id
        registry = "${dubbo.registry.id}")//注册中心id
@Slf4j
public class HelloServiceImpl implements IHelloService {
    
    @Override
    public String hello(String name) {
        log.info("dubbo提供者，参数name:{}", name);
        return "hello " + name + ",this is a dubbo provider!";
    }

}
```

说明下：这里的@Service是包路径com.alibaba.dubbo.config.annotation.Service下的注解类，其指定了接口版本、协议id、注册中心id等基本信息。这里注意还是版本号有用，因为会一个接口多版本共存问题，所以一般上都会设置版本信息的。 2.设置配置文件信息，添加dubbo相关信息，比如注册中心类型，地址等。

```java
# 应用名称 便于识别
dubbo.application.id=spring-boot-dubbo-provider
dubbo.application.name=spring-boot-dubbo-provider

server.port=8686


# 设置版本
demo.service.version=1.0.0

#协议 可选dubbo redis、http、thrift等
dubbo.protocol.id=dubbo
dubbo.protocol.name=dubbo
dubbo.protocol.port=20880

#设置扫描路径 被注解@service和@Reference 等
dubbo.scan.basePackages=cn.lqdev.learning.springboot.dubbo.provider.service

# 注册中心配置
dubbo.registry.id=okong-registry
#注册中心类型 这里使用redis作为注册中心
# zookeeper://127.0.0.1:2181
dubbo.registry.address=redis://127.0.0.1:6379
# 设置用户名密码 若有的话
#dubbo.registry.username=oKong
#dubbo.registry.password=oKong
# 设置redis参数
# 连接池中的最大空闲连接
dubbo.registry.parameters.max.idle=8
# 连接池最大连接数（使用负值表示没有限制）
dubbo.registry.parameters.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制）
dubbo.registry.parameters.max-wait=-1
# 连接池中的最大空闲连接
dubbo.registry.parameters.max-idle=8
# 连接池中的最小空闲连接
dubbo.registry.parameters.min-idle=0
```

注意：这里为了方便，直接使用了Redis作为了注册中心。对于redis连接相关配置参数，可以通过dubbo.registry.parameters.xxx的形式来进行设置，由于parameters是个Map对象，所以添加的key是不会进行大小写转换的，填写了什么就是什么。具体的registry配置对象，可以查看com.alibaba.dubbo.config.RegistryConfig类。而对于redis相关参数配置，可以查看com.alibaba.dubbo.registry.redis.RedisRegistry类。

其他的注册中心，也是类似的，大家可在包com.alibaba.dubbo.registry找到都要的注册中心配置类

3.启动类编写。

DubboProviderApplication.java

```java
/**
 * dubbo-提供者
 * @author oKong
 *
 */
@SpringBootApplication
@Slf4j
public class DubboProviderApplication {

    public static void main(String[] args) throws Exception {
        //由于dubbo提供者只是单纯提供服务的 可以为一个非web环境
        new SpringApplicationBuilder(DubboProviderApplication.class).web(false).run(args);
        log.info("spring-boot-dubbo-provider启动!");
    }

}
```

4.启动应用，可以访问下redis服务，可以看见已经有服务列表信息了。

### 服务消费者

创建spring-boot-dubbo-consumer工程。 0.引入pom依赖

```java
    <!-- 引入api -->
        <dependency>
            <groupId>cn.lqdev.learning</groupId>
            <artifactId>spring-boot-dubbo-api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
        <!-- 引入dubbo依赖 -->
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>0.1.1</version>
        </dependency>
        <!-- 引入redis作为注册中心 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

1.配置文件添加注册中心及服务版本相关信息

```java
# 应用名称 便于识别
dubbo.application.id=spring-boot-dubbo-consumer
dubbo.application.name=spring-boot-dubbo-consumer

server.port=9696

#设置扫描路径 被注解@service和@Reference 等
dubbo.scan.basePackages=cn.lqdev.learning.springboot.dubbo.consumer

# 注册中心配置
dubbo.registry.id=okong-registry
#注册中心类型 这里使用redis作为注册中心
# zookeeper://127.0.0.1:2181
dubbo.registry.address=redis://127.0.0.1:6379
# 设置用户名密码 若有的话
#dubbo.registry.username=oKong
#dubbo.registry.password=oKong
# 设置redis参数
# 连接池中的最大空闲连接
dubbo.registry.parameters.max.idle=8
# 连接池最大连接数（使用负值表示没有限制）
dubbo.registry.parameters.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制）
dubbo.registry.parameters.max-wait=-1
# 连接池中的最大空闲连接
dubbo.registry.parameters.max-idle=8
# 连接池中的最小空闲连接
```

2.启动类编写

DubboConsumerApplication.java

```java
/**
 * dubbo-消费者实例
 * @author oKong
 *
 */
@SpringBootApplication
@Slf4j
public class DubboConsumerApplication {
    
    public static void main(String[] args) throws Exception {
        SpringApplication.run(DubboConsumerApplication.class, args);
        log.info("spring-boot-dubbo-consumer启动!");
    }
}
```

3.编写一个restapi接口服务，进行服务调用。

```java
/**
 * 调用实例
 * @author oKong
 *
 */
@RestController
@Slf4j
public class DemoController {

    /**
     * 申明为一个reference，其实就是设置一个bean类了，
     * 将原来xml配置替换成注解而已
     * <dubbo:reference id=“xxxService” interface=“com.xxx.XxxService” />
     */
    @Reference(version = "1.0.0")
    IHelloService helloService;
    
    @GetMapping("/hello")
    public String hello(String name) {
        log.info("调用提供者服务，参数name：{}", name);
        return helloService.hello(name);
    }
}
```

4.启动应用，访问：http://127.0.0.1:9696/hello?name=oKong ，可以看见服务调用成功了。

### 监控后台

> 官方监控默认支持了zookeeper。而且官方文档也说了，对于redis桥接实现只为开源版本提供，其可靠性依赖于 Redis 本身的可靠性。建议大家还是使用zookeeper吧,redis还是作为缓存使用吧。

监控台地址：https://github.com/apache/incubator-dubbo-ops 大家可自行安装说明进行编译运行下。

develop分支是开发版本和老版本完全不同了

master是老版本的dubbo-admin (个人还是喜欢这个)

```java
git clone -b master https://github.com/apache/incubator-dubbo-ops.git

mvn clean package 

修改dubbo-admin中的application.properties的zookeeper地址

然后启动

java -jar dubbo-admin-0.0.1-SNAPSHOT.jar

访问 http://localhost:7001 账号密码：root root
```

![](/images/20181007162625.png)

### 参考资料

1. http://jm.taobao.org/2018/06/13/%E5%BA%94%E7%94%A8/
2. http://dubbo.apache.org/zh-cn/docs/user/preface/architecture.html

### 总结

本章节主要介绍了dubbo的集成和简单的使用。具体其他的使用其实和原先是一样的，并没有什么区别。建议大家还是去看看官方文档，目前改版后内容丰富多了，干货很多，建议还是去看看。这下终于是中文版的了，看的不头疼了，⊙﹏⊙‖∣

例子:https://github.com/ciweigg2/springboot-dubbo.git
