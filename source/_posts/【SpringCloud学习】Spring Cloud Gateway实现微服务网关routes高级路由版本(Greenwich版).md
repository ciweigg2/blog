title: 【SpringCloud学习】Spring Cloud Gateway实现微服务网关routes高级路由版本(Greenwich版)
date: 2019-07-21 10:49:36
tags: [springcloud]
categories: [springcloud]
---
### Spring Cloud Gateway

Spring Cloud Gateway 是 Spring Cloud 的一个全新项目，该项目是基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。

Spring Cloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是替代 Netflix Zuul，其不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/指标，和限流。

<!--more-->

### 相关概念

* Route（路由）：这是网关的基本构建块。它由一个 ID，一个目标 URI，一组断言和一组过滤器定义。如果断言为真，则路由匹配。
* Predicate（断言）：这是一个 Java 8 的 Predicate。输入类型是一个 ServerWebExchange。我们可以使用它来匹配来自 HTTP 请求的任何内容，例如 headers 或参数。
* Filter（过滤器）：这是org.springframework.cloud.gateway.filter.GatewayFilter的实例，我们可以使用它修改请求和响应。

### 工作流程

![](/images/spring-cloud-gateway20190721.png)

客户端向 Spring Cloud Gateway 发出请求。如果 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。 过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑。

Spring Cloud Gateway 的特征：

* 基于 Spring Framework 5，Project Reactor 和 Spring Boot 2.0
* 动态路由
* Predicates 和 Filters 作用于特定路由
* 集成 Hystrix 断路器
* 集成 Spring Cloud DiscoveryClient
* 易于编写的 Predicates 和 Filters
* 限流
* 路径重写

### 快速上手

Spring Cloud Gateway 网关路由有两种配置方式：

* 在配置文件 yml 中配置

* 通过@Bean自定义 RouteLocator，在启动主类 Application 中配置

这两种方式是等价的，建议使用 yml 方式进配置。

添加项目需要使用的依赖包

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

Spring Cloud Gateway 是使用 netty+webflux 实现因此不需要再引入 web 模块。

我们先来测试一个最简单的请求转发。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: neo_route
        uri: http://www.ityouknow.com
        predicates:
        - Path=/spring-cloud
```

各字段含义如下：

* id：我们自定义的路由 ID，保持唯一
* uri：目标服务地址
* predicates：路由条件，Predicate 接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（比如：与，或，非）。
* filters：过滤规则，本示例暂时没用。

上面这段配置的意思是，配置了一个 id 为 neo_route 的路由规则，当访问地址 http://localhost:8080/spring-cloud时会自动转发到地址：http://www.ityouknow.com/spring-cloud。配置完成启动项目即可在浏览器访问进行测试，当我们访问地址http://localhost:8080/spring-cloud 时会展示页面展示如下：

![](/images/spring-cloud-gateway120190721.png)

证明页面转发成功。

转发功能同样可以通过代码来实现，我们可以在启动类 GateWayApplication 中添加方法 customRouteLocator() 来定制转发规则。

```java
@SpringBootApplication
public class GateWayApplication {

	public static void main(String[] args) {
		SpringApplication.run(GateWayApplication.class, args);
	}

	@Bean
	public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
		return builder.routes()
				.route("path_route", r -> r.path("/about")
						.uri("http://ityouknow.com"))
				.build();
	}

}
```

上面配置了一个 id 为 path_route 的路由，当访问地址http://localhost:8080/about时会自动转发到地址：http://www.ityouknow.com/about和上面的转发效果一样，只是这里转发的是以项目地址/about格式的请求地址。

上面两个示例中 uri 都是指向了我的个人网站，在实际项目使用中可以将 uri 指向对外提供服务的项目地址，统一对外输出接口。

以上便是 Spring Cloud Gateway 最简单的两个请求示例，Spring Cloud Gateway 还有更多实用的功能接下来我们一一介绍。

### 路由规则

Spring Cloud Gateway 的功能很强大，我们仅仅通过 Predicates 的设计就可以看出来，前面我们只是使用了 predicates 进行了简单的条件匹配，其实 Spring Cloud Gataway 帮我们内置了很多 Predicates 功能。

Spring Cloud Gateway 是通过 Spring WebFlux 的 HandlerMapping 做为底层支持来匹配到转发路由，Spring Cloud Gateway 内置了很多 Predicates 工厂，这些 Predicates 工厂通过不同的 HTTP 请求参数来匹配，多个 Predicates 工厂可以组合使用。

#### Predicate 介绍

Predicate 来源于 Java 8，是 Java 8 中引入的一个函数，Predicate 接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（比如：与，或，非）。可以用于接口请求参数校验、判断新老数据是否有变化需要进行更新操作。

在 Spring Cloud Gateway 中 Spring 利用 Predicate 的特性实现了各种路由匹配规则，有通过 Header、请求参数等不同的条件来进行作为条件匹配到对应的路由。网上有一张图总结了 Spring Cloud 内置的几种 Predicate 的实现。

![](/images/spring-cloud-gateway320190721.png)

说白了 Predicate 就是为了实现一组匹配规则，方便让请求过来找到对应的 Route 进行处理，接下来我们接下 Spring Cloud GateWay 内置几种 Predicate 的使用。

#### 通过时间匹配

Predicate 支持设置一个时间，在请求进行转发的时候，可以通过判断在这个时间之前或者之后进行转发。

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: time_route
          uri: http://ityouknow.com
          predicates:
          - After=2018-01-20T06:06:06+08:00[Asia/Shanghai]
```

Spring 是通过 ZonedDateTime 来对时间进行的对比，ZonedDateTime 是 Java 8 中日期时间功能里，用于表示带时区的日期与时间信息的类，ZonedDateTime 支持通过时区来设置时间，中国的时区是：Asia/Shanghai。

After Route Predicate 是指在这个时间之后的请求都转发到目标地址。上面的示例是指，请求时间在 2018年1月20日6点6分6秒之后的所有请求都转发到地址http://ityouknow.com。+08:00是指时间和UTC时间相差八个小时，时间地区为Asia/Shanghai。

添加完路由规则之后，访问地址http://localhost:8080会自动转发到http://ityouknow.com。

Before Route Predicate 刚好相反，在某个时间之前的请求的请求都进行转发。我们把上面路由规则中的 After 改为 Before，如下spring:
  cloud:
    gateway:
      routes:
        - id: after_ id: after_route
          uri: http://ityouknow.com
          predicates:
          - Before=2018-01-20T06:06:06+08:00[Asia/Shanghai]
```

就表示在这个时间之前可以进行路由，在这时间之后停止路由，修改完之后重启项目再次访问地址http://localhost:8080，页面会报 404 没有找到地址。

除过在时间之前或者之后外，Gateway 还支持限制路由请求在某一个时间段范围内，可以使用 Between Route Predicate 来实现。

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: after_route
          uri: http://ityouknow.com
          predicates:
            - Between=2018-01-20T06:06:06+08:00[Asia/Shanghai], 2019-01-20T06:06:06+08:00[Asia/Shanghai]
```

这样设置就意味着在这个时间段内可以匹配到此路由，超过这个时间段范围则不会进行匹配。通过时间匹配路由的功能很酷，可以用在限时抢购的一些场景中。

#### 通过 Cookie 匹配

Cookie Route Predicate 可以接收两个参数，一个是 Cookie name ,一个是正则表达式，路由规则会通过获取对应的 Cookie name 值和正则表达式去匹配，如果匹配上就会执行路由，如果没有匹配上则不执行。

```yaml
spring:
  cloud:
    gateway:
      routes:
       - id: cookie_route
         uri: http://ityouknow.com
         predicates:
         - Cookie=ityouknow, kee.e
```

使用 curl 测试，命令行输入:

```
curl http://localhost:8080 --cookie "ityouknow=kee.e"
```

则会返回页面代码，如果去掉--cookie "ityouknow=kee.e"，后台汇报 404 错误。

#### 通过 Header 属性匹配

Header Route Predicate 和 Cookie Route Predicate 一样，也是接收 2 个参数，一个 header 中属性名称和一个正则表达式，这个属性值和正则表达式匹配则执行。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: http://ityouknow.com
        predicates:
        - Header=X-Request-Id, \d+
```

使用 curl 测试，命令行输入:

```
curl http://localhost:8080  -H "X-Request-Id:666666" 
```

则返回页面代码证明匹配成功。将参数-H "X-Request-Id:666666"改为-H "X-Request-Id:neo"再次执行时返回404证明没有匹配。

#### 通过 Host 匹配

Host Route Predicate 接收一组参数，一组匹配的域名列表，这个模板是一个 ant 分隔的模板，用.号作为分隔符。它通过参数中的主机地址作为匹配规则。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: http://ityouknow.com
        predicates:
        - Host=**.ityouknow.com
```

使用 curl 测试，命令行输入:

```
curl http://localhost:8080  -H "Host: www.ityouknow.com" 
curl http://localhost:8080  -H "Host: md.ityouknow.com" 
```

经测试以上两种 host 均可匹配到 host_route 路由，去掉 host 参数则会报 404 错误。

#### 通过请求方式匹配

可以通过是 POST、GET、PUT、DELETE 等不同的请求方式来进行路由。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: http://ityouknow.com
        predicates:
        - Method=GET
```

使用 curl 测试，命令行输入:

```
curl 默认是以 GET 的方式去请求
curl http://localhost:8080
```

测试返回页面代码，证明匹配到路由，我们再以 POST 的方式请求测试。

```
curl 默认是以 GET 的方式去请求
curl -X POST http://localhost:8080
```

返回 404 没有找到，证明没有匹配上路由

#### 通过请求路径匹配

Path Route Predicate 接收一个匹配路径的参数来判断是否走路由。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: http://ityouknow.com
        predicates:
        - Path=/foo/{segment}
```

如果请求路径符合要求，则此路由将匹配，例如：/foo/1 或者 /foo/bar。

使用 curl 测试，命令行输入:

```
curl http://localhost:8080/foo/1
curl http://localhost:8080/foo/xx
curl http://localhost:8080/boo/xx
```

经过测试第一和第二条命令可以正常获取到页面返回值，最后一个命令报404，证明路由是通过指定路由来匹配。

#### 通过请求参数匹配

Query Route Predicate 支持传入两个参数，一个是属性名一个为属性值，属性值可以是正则表达式。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: http://ityouknow.com
        predicates:
        - Query=smile
```

这样配置，只要请求中包含 smile 属性的参数即可匹配路由。

使用 curl 测试，命令行输入:

```
curl localhost:8080?smile=x&id=2
```

经过测试发现只要请求汇总带有 smile 参数即会匹配路由，不带 smile 参数则不会匹配。

还可以将 Query 的值以键值对的方式进行配置，这样在请求过来时会对属性值和正则进行匹配，匹配上才会走路由。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: http://ityouknow.com
        predicates:
        - Query=keep, pu.
```

这样只要当请求中包含 keep 属性并且参数值是以 pu 开头的长度为三位的字符串才会进行匹配和路由。

使用 curl 测试，命令行输入:

```
curl localhost:8080?keep=pub
```

测试可以返回页面代码，将 keep 的属性值改为 pubx 再次访问就会报 404,证明路由需要匹配正则表达式才会进行路由。

#### 通过请求 ip 地址进行匹配

Predicate 也支持通过设置某个 ip 区间号段的请求才会路由，RemoteAddr Route Predicate 接受 cidr 符号(IPv4 或 IPv6 )字符串的列表(最小大小为1)，例如 192.168.0.1/16 (其中 192.168.0.1 是 IP 地址，16 是子网掩码)。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: http://ityouknow.com
        predicates:
        - RemoteAddr=192.168.1.1/24
```

可以将此地址设置为本机的 ip 地址进行测试。

```
curl localhost:8080
```

果请求的远程地址是 192.168.1.10，则此路由将匹配。

#### 组合使用

上面为了演示各个 Predicate 的使用，我们是单个单个进行配置测试，其实可以将各种 Predicate 组合起来一起使用。

例如：

```yaml
spring:
  cloud:
    gateway:
      routes:
       - id: host_foo_path_headers_to_httpbin
        uri: http://ityouknow.com
        predicates:
        - Host=**.foo.org
        - Path=/headers
        - Method=GET
        - Header=X-Request-Id, \d+
        - Query=foo, ba.
        - Query=baz
        - Cookie=chocolate, ch.p
        - After=2018-01-20T06:06:06+08:00[Asia/Shanghai]
```

再来个：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: time_route_before
          uri: http://ityouknow.com
          # 请求时间在 2018年1月20日6点6分6秒之前的所有请求都转发到地址http://ityouknow.com
          predicates:
          - Before=2018-01-20T06:06:06+08:00[Asia/Shanghai]
        - id: time_route_after
          uri: lb://eureka-client-a
          # 请求时间在 2018年1月20日6点6分6秒之后的所有请求都转发到地址eureka-client-a
          predicates:
          - After=2018-01-20T06:06:06+08:00[Asia/Shanghai]
```

根据接口不同匹配到不同的服务

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: nacos_route
          uri: lb://eureka-client-a
          predicates:
          - Path=/nacos/**
        - id: users_route
          uri: lb://eureka-client-b
          predicates:
          - Path=/users/**
```

各种 Predicates 同时存在于同一个路由时，请求必须同时满足所有的条件才被这个路由匹配。

一个请求满足多个路由的谓词条件时，请求只会被首个成功匹配的路由转发

### 限流路由器

限速在高并发场景中比较常用的手段之一，可以有效的保障服务的整体稳定性，Spring Cloud Gateway 提供了基于 Redis 的限流方案。所以我们首先需要添加对应的依赖包spring-boot-starter-data-redis-reactive

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

配置文件中需要添加 Redis 地址和限流的相关配置

```yaml
spring:
  redis:
    host: 127.0.0.1
    password:
    port: 6379
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
      routes:
        - id: requestratelimiter_route
          uri: lb://eureka-client-a
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
                key-resolver: "#{@hostAddrKeyResolver}"
          predicates:
            - Method=GET
```

* filter 名称必须是 RequestRateLimiter

* redis-rate-limiter.replenishRate：允许用户每秒处理多少个请求

* redis-rate-limiter.burstCapacity：令牌桶的容量，允许在一秒钟内完成的最大请求数

* key-resolver：使用 SpEL 按名称引用 bean

下面的限流规则只能存在一个bean 不能同时存在 因为源码中限制只能存在一个bean了呀

#### IP限流

获取请求用户ip作为限流key。

```java
@Bean
public KeyResolver hostAddrKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getRemoteAddress().getHostName());
}
```

#### 用户限流

获取请求用户id作为限流key 必须请求参数中包含userId 否则无法测试的

```java
@Bean
public KeyResolver userKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("userId"));
}
```

#### 接口限流

获取请求地址的uri作为限流key。

```java
@Bean
KeyResolver apiKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getPath().value());
}
```

### 重试路由器

RetryGatewayFilter 是 Spring Cloud Gateway 对请求重试提供的一个 GatewayFilter Factory

配合ClientA和Feign例子使用

配置示例：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: retry_test
          uri: lb://spring-cloud-feign
          predicates:
            - Path=/hello/**
          filters:
            - name: Retry
              args:
                retries: 3
                statuses: INTERNAL_SERVER_ERROR
```

Retry GatewayFilter 通过这四个参数来控制重试机制： retries, statuses, methods, 和 series。

* retries：重试次数，默认值是3次
* statuses：HTTP 的状态返回码，取值请参考：org.springframework.http.HttpStatus
* methods：指定哪些方法的请求需要进行重试逻辑，默认值是 GET 方法，取值参考：org.springframework.http.HttpMethod
* series：一些列的状态码配置，取值参考：org.springframework.http.HttpStatus.Series。符合的某段状态码才会进行重试逻辑，默认值是 SERVER_ERROR，值是5，也就是 5XX(5开头的状态码)，共有5个值。

高级功能参考：http://cxytiandi.com/blog/detail/20545

### 权重路由

```yaml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
      - id: service1_prod
        uri: http://localhost:8081
        predicates:
        - Path=/test
        - Weight=service1, 90
      - id: service1_canary
        uri: http://localhost:8082
        predicates:
        - Path=/test
        - Weight=service1, 10
```

demo：https://github.com/ciweigg2/spring-cloud-examples/tree/spring-cloud-nacos/spring-cloud-gateway