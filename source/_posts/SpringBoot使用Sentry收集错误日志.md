cover: http://ciwei2.cn-sh2.ufileos.com/115.jpg
title: SpringBoot使用Sentry收集错误日志
date: 2018-07-25 22:12:16
tags: [sentry]
categories: [日志框架]
---
### SpringBoot中怎么使用Sentry的
<!--more-->
```java
		<dependency>
			<groupId>io.sentry</groupId>
			<artifactId>sentry-logback</artifactId>
			<version>1.7.5</version>
		</dependency>
```

#### 添加logback.xml
```java
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
<!-- Configure the Console appender -->
<appender name="Console" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
</appender>

<!-- Configure the Sentry appender, overriding the logging threshold to the WARN level -->
<appender name="Sentry" class="io.sentry.logback.SentryAppender">
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
        <level>WARN</level>
    </filter>
</appender>

<!-- Enable the Console and Sentry appenders, Console is provided as an example
     of a non-Sentry logger that is set to a different logging threshold -->
<root level="INFO">
    <appender-ref ref="Console" />
    <appender-ref ref="Sentry" />
</root>
</configuration>
```

#### 添加sentry.properties
//在sentry服务器中创建的项目中有settings中的Clients keys(DSN)
dsn=xxx
tags=tag1:value1,tag2:value2

```java
	@GetMapping(value="test")
	public void test(){
		log.info("测试");
		log.error("错误");
		log.debug("debug");
		int a = 1/0;
	}
```
调用接口会发现日志和报错都输出在sentry项目中了呢