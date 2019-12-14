title: 【SpringCloud学习】Spring Cloud Gateway实现微服务网关discovery最基础版本(Greenwich版)
date: 2019-07-20 12:18:21
tags: [springcloud]
categories: [springcloud]
---
## Spring Cloud Gateway实现微服务网关

### Spring Cloud Gateway简介
该项目提供了一个建立在Spring Ecosystem之上的API网关，包括：Spring 5，Spring Boot 2和Project Reactor。Spring Cloud Gateway旨在提供一种简单而有效的方式来路由到API，
并为他们提供横切关注点，例如：安全性，监控/指标和弹性。

<!--more-->

### 架构图

![](https://ciweigg2.github.io/images/2019/gateway_diagram.png)
客户端向Spring Cloud Gateway发出请求。如果网关处理程序映射确定请求与路由匹配，则将其发送到网关Web处理程序。此处理程序运行通过特定于请求的过滤器链发送请求。
滤波器被虚线划分的原因是滤波器可以在发送代理请求之前或之后执行逻辑。执行所有“预”过滤器逻辑，然后进行代理请求。在发出代理请求之后，执行“post”过滤器逻辑。
### 前期准备工作
* 启动[spring-cloud-eureka](https://github.com/ciweigg2/spring-cloud-examples/tree/master/spring-cloud-eureka)
* 启动[spring-cloud-clientA](https://github.com/ciweigg2/spring-cloud-examples/tree/master/spring-cloud-clientA)
* 启动[spring-cloud-clientB](https://github.com/ciweigg2/spring-cloud-examples/tree/master/spring-cloud-clientB)

### gateway服务搭建

maven依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

application.properties配置

```properties
server.port=8007
spring.application.name=spring-cloud-gateway
spring.cloud.gateway.discovery.locator.enabled=true
spring.cloud.gateway.discovery.locator.lower-case-service-id=true

eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
```

网关支持跨域

注释application.properties的配置使用如下配置

application.yml

```yaml
server:
  port: 8007
spring:
  application:
    name: spring-cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #是否与服务注册于发现组件进行结合，通过 serviceId 转发到具体的服务实例。默认为 false，设为 true 便开启通过服务中心的自动根据 serviceId 创建路由的功能。
          lower-case-service-id: true
      globalcors: #网关支持跨域问题
        cors-configurations:
          '[/**]':
            allowedOrigins: "*" # * 表示所有
            allowedMethods:
              - GET
              - POST
              - PUT
```

启动类案例

```java
@EnableEurekaClient
@SpringBootApplication
public class SpringCloudGatewayApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudGatewayApplication.class, args);
	}

}
```

启动项目[spring-cloud-gateway](./spring-cloud-gateway)

访问 http://localhost:8007/eureka-client-a/hello/zwd ,可以从服务a控制台看到调用成功。

访问 http://localhost:8007/eureka-client-b/hello/zwd ,可以从服务b控制台看到调用成功。
