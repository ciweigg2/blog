title: 【SpringCloud学习】Ncos实现配置中心(Greenwich版)
date: 2019-07-20 18:30:38
tags: [springcloud,nacos]
categories: [springcloud]
---
### 添加依赖

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
            <version>0.9.0.RELEASE</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

<!--more-->

### 添加nacos注册中心

application.properties

```properties
spring.cloud.nacos.discovery.server-addr=115.220.10.37:8848
```

### 添加bootstrap.properties

为什么使用bootstrap.properties 因为bootstrap.properties加载优先级会高于bean的加载优先级 否则会读取不到配置的

根据如下配置在nacos控制台添加配置 这篇文章主要介绍了多配置文件的使用呀

```properties
#config加载的优先级高
spring.cloud.nacos.config.server-addr=127.0.0.1:8848

# 默认读取properties spring.cloud.nacos.config.file-extension配置的
# Nacos 控制台添加配置：
#   Data ID：app.properties
#   Group：multi-data-ids
#   配置内容：app.user.cache=false
spring.cloud.nacos.config.ext-config[0].data-id=app.properties
spring.cloud.nacos.config.ext-config[0].group=multi-data-ids
spring.cloud.nacos.config.ext-config[0].refresh=true

# 默认读取properties spring.cloud.nacos.config.file-extension配置的
# 1. 本地安装 Redis
# 2. Nacos 控制台添加配置：
#   Data ID：redis.properties
#   Group：multi-data-ids
#   配置内容示例：
#       spring.redis.host=localhost
#       spring.redis.port=6379
#       spring.redis.password=20190101
#       spring.redis.timeout=5000
#       spring.redis.max-idle=5
#       spring.redis.max-active=10
#       spring.redis.max-wait=3000
#       spring.redis.test-on-borrow=false
spring.cloud.nacos.config.ext-config[1].data-id=redis.properties
spring.cloud.nacos.config.ext-config[1].group=multi-data-ids

# 默认读取properties spring.cloud.nacos.config.file-extension配置的
# 1. 对象操作呀
# 2. Nacos 控制台添加配置：
#   Data ID：user.yml
#   Group：multi-data-ids
#   配置内容示例：
#       user:
#           id: 12323213123323123
spring.cloud.nacos.config.ext-config[2].data-id=user.yml
spring.cloud.nacos.config.ext-config[2].group=multi-data-ids
spring.cloud.nacos.config.ext-config[2].refresh=true
spring.cloud.nacos.config.file-extension=yml
```

### 测试

如果开启了自动刷新 需要在使用@Value的类中添加@RefreshScope

```java
@RestController
@RefreshScope
public class ConsumerController {

    @Value("${app.user.cache}")
    private boolean cache;

    @Autowired
    RestTemplate restTemplate;

    @Autowired
    private RedisTemplate redisTemplate;

    @Autowired
    private User user;

    @GetMapping(value = "ribbon")
    public String add() {
        redisTemplate.opsForSet().add("spring-cloud-nacos" ,"测试一下呀");
        return restTemplate.getForEntity("http://eureka-client-a/hello/zwd",String.class).getBody();
    }
}
```

### 对象的使用

都是用@Value获取的

```java
@Data
@Component
@RefreshScope
public class User implements Serializable {

	@Value("${user.id}")
	private String id;

}
```

demo：https://github.com/ciweigg2/spring-cloud-examples/tree/spring-cloud-nacos/spring-cloud-nacos-multi-config