title: springboot使用actuator动态修改日志级别
date: 2019-12-06 13:48:13
tags: [springboot,logback]
categories: [综合]
---
### Spring Boot 依赖与配置

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

<!--more-->

#### Actuator 配置

```yml
management:
  endpoints:
    web:
      exposure:
        include: loggers
```

### 初始化配置日志级别

可选的日志级别

* OFF
* ERROR
* WARN
* INFO
* DEBUG
* TRACE

### 设置全局日志级别

```properties
logging.level.root=INFO
```

设置特定 Package/Class 日志级别

```properties
logging.level.<Package/Class>=INFO
```

> 示例：logging.level.com.anoyi=INFO

### 运行时配置日志级别

#### 查看所有 package / class 日志级别的配置

```bash
curl http://127.0.0.1:8080/actuator/loggers
```

#### 查看单个 package / class 日志级别的配置

```bash
# 用法
curl http://127.0.0.1:8080/actuator/loggers/<Package/Class>

# 示例
curl http://127.0.0.1:8080/actuator/loggers/com.anoyi
```

#### 动态修改日志级别

```bash
# 用法
curl -X POST \
  http://localhost:8080/actuator/loggers/<Package/Class> \
  -d '{"configuredLevel":"<LEVEL>"}'

# 示例
curl -X POST \
  http://localhost:8080/actuator/loggers/com.anoyi \
  -d '{"configuredLevel":"DEBUG"}'
```