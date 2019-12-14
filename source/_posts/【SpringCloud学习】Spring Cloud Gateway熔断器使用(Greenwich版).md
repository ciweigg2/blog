title: 【SpringCloud学习】Spring Cloud Gateway熔断器使用(Greenwich版)
date: 2019-07-21 20:42:54
tags: [springcloud]
categories: [springcloud]
---
本文需要的demo：

服务Provider：[ClientA](https://github.com/ciweigg2/spring-cloud-examples/tree/spring-cloud-nacos/spring-cloud-clientA)

服务Consumer：[Feign](https://github.com/ciweigg2/spring-cloud-examples/tree/spring-cloud-nacos/spring-cloud-feign)

网关：[Gateway](https://github.com/ciweigg2/spring-cloud-examples/tree/spring-cloud-nacos/spring-cloud-gateway)

<!--more-->

maven需要添加：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

具体重试代码：

```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
      routes:
        - id: hystrix_route
          uri: lb://spring-cloud-feign
          predicates:
            - Path=/hello/**
          filters:
            - StripPrefix=0
            - name: Hystrix
              args:
                name: fallbackcmd
                fallbackUri: forward:/fallback
# 超时熔断 设置为5s 如果请求在5s内还没有响应直接进入熔断逻辑
hystrix.command.fallbackcmd.execution.isolation.thread.timeoutInMilliseconds: 5000
```

fallbackUri: forward:/fallback会跳转到gateway的接口fallback

熔断的代码：

```java
@RestController
public class HystrixController {

	@GetMapping("/fallback")
	public Map<String,Object> fallback() {
		Map<String,Object> map = new HashMap<String,Object>();
		map.put("Code",200);
		map.put("Message","请稍后再试");
		return map;
	}

}
```

访问：http://localhost:8007/hello/ciwei在feign打个断点5s后会自动熔断的