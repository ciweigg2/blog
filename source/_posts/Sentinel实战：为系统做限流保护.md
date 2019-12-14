cover: http://ciwei2.cn-sh2.ufileos.com/96.jpg
title: Sentinel实战：为系统做限流保护
date: 2019-01-20 17:44:41
tags: [dubbo,sentinel]
categories: [综合]
---
### 本demo介绍

> 集成了nacos配置中心做sentinel数据的持久化集成了普通限流和热点限流

<!--more-->

PS：如果你不想对原有的业务代码进行侵入，也可以通过注解 SentinelResource 来进行资源埋点。

## 定义规则

定义完资源后，就可以来定义限流的规则了，但是我们需要对流控规则做个详细的了解，以便更好的进行限流的操作，流控的规则对应的是 FlowRule。

一条FlowRule有以下几个重要的属性组成：

- resource: 规则的资源名
- grade: 限流阈值类型，qps 或线程数
- count: 限流的阈值
- limitApp: 被限制的应用，授权时候为逗号分隔的应用集合，限流时为单个应用
- strategy: 基于调用关系的流量控制
- controlBehavior：流控策略

前三个属性比较好理解，最后三个比较难理解，让我们来详细看下最后三个属性：

### limitApp

首先让我们来看下limitApp，从字面上看是指要限制哪个应用的意思，主要是用于根据调用方进行流量控制。

他有三种情况可以选择：

- default

表示不区分调用者，来自任何调用者的请求都将进行限流统计。

- {some_origin_name}

表示针对特定的调用者，只有来自这个调用者的请求才会进行流量控制。

例如：资源 `NodeA` 配置了一条针对调用者 **caller1** 的规则，那么当且仅当来自 **caller1** 对 `NodeA` 的请求才会触发流量控制。

- other

表示除 {some_origin_name} 以外的其余调用方的流量进行流量控制。

例如：资源 `NodeA` 配置了一条针对调用者 **caller1** 的限流规则，同时又配置了一条调用者为 **other** 的规则，那么任意来自非 **caller1** 对 `NodeA` 的调用，都不能超过 **other** 这条规则定义的阈值。

### strategy

基于调用关系的流量控制，也有三种情况可以选择：

- STRATEGY_DIRECT

根据调用方进行限流。ContextUtil.enter(resourceName, origin) 方法中的 origin 参数标明了调用方的身份。

如果 strategy 选择了DIRECT ，则还需要根据限流规则中的 limitApp 字段根据调用方在不同的场景中进行流量控制，包括有：”所有调用方“、”特定调用方origin“、”除特定调用方origin之外的调用方“。

- STRATEGY_RELATE

根据关联流量限流。当两个资源之间具有资源争抢或者依赖关系的时候，这两个资源便具有了关联，可使用关联限流来避免具有关联关系的资源之间过度的争抢。

比如对数据库同一个字段的读操作和写操作存在争抢，读的速度过高会影响写得速度，写的速度过高会影响读的速度。

举例来说：read_db 和 write_db 这两个资源分别代表数据库读写，我们可以给 read_db 设置限流规则来达到写优先的目的：设置 FlowRule.strategy 为 RuleConstant.STRATEGY_RELATE，同时设置 FlowRule.refResource 为 write_db。这样当写库操作过于频繁时，读数据的请求会被限流。


- STRATEGY_CHAIN

根据调用链路入口限流。假设来自入口 Entrance1 和 Entrance2 的请求都调用到了资源 NodeA，Sentinel 允许根据某个入口的统计信息对资源进行限流。

举例来说：我们可以设置 FlowRule.strategy 为 RuleConstant.CHAIN，同时设置 FlowRule.refResource 为 Entrance1 来表示只有从入口 Entrance1 的调用才会记录到 NodeA 的限流统计当中，而对来自 Entrance2 的调用可以放行。

### controlBehavior

流控策略，主要是发生拦截后具体的流量整形和控制策略，目前有三种策略，分别是：

- CONTROL_BEHAVIOR_DEFAULT

这种方式是：**直接拒绝**，该方式是默认的流量控制方式，当 qps 超过任意规则的阈值后，新的请求就会被立即拒绝，拒绝方式为抛出FlowException。

这种方式适用于对系统处理能力确切已知的情况下，比如通过压测确定了系统的准确水位。

- CONTROL_BEHAVIOR_WARM_UP

这种方式是：**排队等待** ，又称为 **冷启动**。该方式主要用于当系统长期处于低水位的情况下，流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮。

通过"冷启动"，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮的情况。

- CONTROL_BEHAVIOR_RATE_LIMITER

这种方式是：**慢启动**，又称为 **匀速器模式**。这种方式严格控制了请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的是漏桶算法。

这种方式主要用于处理间隔性突发的流量，例如消息队列。想象一下这样的场景，在某一秒有大量的请求到来，而接下来的几秒则处于空闲状态，我们希望系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。

具体的 FlowRule 可以用下面这张图表示：

![](/images/flow-rule-factors.png)

```java
git clone -b dubbo-sentinel https://github.com/ciweigg2/springboot-dubbo-nacos-zipkin.git
```

### 集成限流

* 引入maven(因为使用了nacos持久化数据和注解方式索引引用的比较多的呀)
```java
            <!--sentinet限流配置-->
            <dependency>
                <groupId>com.alibaba.csp</groupId>
                <artifactId>sentinel-dubbo-adapter</artifactId>
                <version>1.4.2-SNAPSHOT</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba.csp</groupId>
                <artifactId>sentinel-transport-simple-http</artifactId>
                <version>1.4.2-SNAPSHOT</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba.csp</groupId>
                <artifactId>sentinel-parameter-flow-control</artifactId>
                <version>1.4.2-SNAPSHOT</version>
            </dependency>

            <!--sentinel使用nacos存储配置-->
            <dependency>
                <groupId>com.alibaba.csp</groupId>
                <artifactId>sentinel-datasource-extension</artifactId>
                <version>1.4.2-SNAPSHOT</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba.csp</groupId>
                <artifactId>sentinel-datasource-nacos</artifactId>
                <version>1.4.2-SNAPSHOT</version>
            </dependency>

            <!--aop-->
            <dependency>
                <groupId>com.alibaba.csp</groupId>
                <artifactId>sentinel-annotation-aspectj</artifactId>
                <version>1.4.2-SNAPSHOT</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-aop</artifactId>
                <version>2.1.2.RELEASE</version>
            </dependency>
```

### 设置流量规则

```java
	/**
	 * 添加流量控制配置参数
	 */
	public static void setFlowRule() throws Exception {
		final String remoteAddress = "118.184.218.184:8848";
		final String groupId = "Sentinel:Demo";
		final String dataId = "com.alibaba.csp.sentinel.demo.flow.rule";
		final String rule = "[\n" +
				"  {\n" +
				"    \"resource\": \"FlowRuleA\",\n" +
				"    \"controlBehavior\": 0,\n" +
				"    \"count\": 5.0,\n" +
				"    \"grade\": 1,\n" +
				"    \"limitApp\": \"default\",\n" +
				"    \"strategy\": 0\n" +
				"  },\n" +
				"  {\n" +
				"    \"resource\": \"FlowRuleB\",\n" +
				"    \"controlBehavior\": 0,\n" +
				"    \"count\": 5.0,\n" +
				"    \"grade\": 1,\n" +
				"    \"limitApp\": \"default\",\n" +
				"    \"strategy\": 0\n" +
				"  }\n" +
				"]";
		ConfigService configService = NacosFactory.createConfigService(remoteAddress);
		log.info(String.valueOf(configService.publishConfig(dataId, groupId, rule)));
	}
```

### 配置注解加载资源哟

```java
@Configuration
public class AopConfiguration {

    @Bean
    public SentinelResourceAspect sentinelResourceAspect() {
        return new SentinelResourceAspect();
    }

}
```

### 配置流量初始化数据

```java
	/**
     * 初始化流量规则
     */
    public void loadFlowRule(){
        final String dataId = "com.alibaba.csp.sentinel.demo.flow.rule";
        ReadableDataSource<String, List<FlowRule>> flowRuleDataSource = new NacosDataSource<>(remoteAddress, groupId, dataId,
                source -> JSON.parseObject(source, new TypeReference<List<FlowRule>>() {}));
        FlowRuleManager.register2Property(flowRuleDataSource.getProperty());
        log.info("初始化流量规则成功");
    }
```

### 方法上添加限流设置 熔断降级机制
```java
@Service(version = "${demo.service.version}")
public class DemoServiceImpl implements DemoService {

    @Reference(version = "${demo.service.version}")
    private UserService userService;

    /**
     * The default value of ${dubbo.application.name} is ${spring.application.name}
     */
    @Value("${dubbo.application.name}")
    private String serviceName;

    @Override
//    @SentinelResource(value = "FlowRuleA",blockHandler = "exceptionHandler")
    // 对应的 `handleException` 函数需要位于 `ExceptionUtil` 类中，并且必须为 static 函数.
    @SentinelResource(value = "DegradeRuleA", blockHandler = "exceptionHandler", fallback = "helloFallback")
//    @SentinelResource(value = "FlowRuleA", blockHandler = "handleException", blockHandlerClass = {ExceptionUtil.class})
    public String sayHello(String name) {
        userService.sayName();
        System.out.println(userService.sayName());
        RpcContext rpcContext = RpcContext.getContext();
        System.out.println(String.format("Service [name :%s , port : %d] %s(\"%s\") : Hello,%s",
                serviceName,
                rpcContext.getLocalPort(),
                rpcContext.getMethodName(),
                name,
                name));
        return String.format("[%s] : Hello, %s", serviceName, name);
    }

    // Fallback 函数，函数签名与原函数一致 fallback只有降级的时候才会触发 业务异常不会进入 fallback 逻辑
    public String helloFallback(String name) {
        return String.format("Halooooo %s", name);
    }

    // Block 异常处理函数，参数最后多一个 BlockException，其余与原函数一致.
    public String exceptionHandler(String name, BlockException ex) {
        return "不好意思当前太挤啦，Oops, error occurred at " + name;
    }

}
```

ExceptionUtil
```java
public class ExceptionUtil {

    public static String handleException(String name ,BlockException ex) {
        return "Oops: " + ex.getClass().getCanonicalName();
    }
}
```

### 访问接口测试

http://localhost:8080/sayHello

多次访问会发现被限流进入熔断方法中，主要根据value = "FlowRuleA"和value = "DegradeRuleA"进入不同的自定义熔断方法，具体参考demo中的使用吧

demo所在分支dubbo-sentinel：https://github.com/ciweigg2/springboot-dubbo-nacos-zipkin.git

### 启动需要添加的参数

> 需要启动sentinel和每个服务添加启动参数

```java
java -Dserver.port=8089 -Dcsp.sentinel.dashboard.server=localhost:8089 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard-1.4.1.jar
```

> 服务：

```java
-Djava.net.preferIPv4Stack=true -Dcsp.sentinel.api.port=8721 -Dcsp.sentinel.dashboard.server=localhost:8089 -Dproject.name=dubbo-consumer

-Djava.net.preferIPv4Stack=true -Dcsp.sentinel.api.port=8721 -Dcsp.sentinel.dashboard.server=localhost:8089 -Dproject.name=user-provider2 

-Djava.net.preferIPv4Stack=true -Dcsp.sentinel.api.port=8720 -Dcsp.sentinel.dashboard.server=localhost:8089 -Dproject.name=demo-provider1 
```