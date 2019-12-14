cover: http://ciwei2.cn-sh2.ufileos.com/105.jpg
title: SpringBoot Tomcat多端口启动
date: 2018-08-11 19:50:51
tags: [SpringBoot Tomcat多端口启动]
categories: [综合]
---
SpringBoot一个项目如果想用多个tomcat启动，那要怎么实现呢
<!--more-->

### Spring Boot 1.x.x

```java
@SpringBootApplication
public class EventbusApplication {

    public static void main(String[] args) {
        SpringApplication.run(EventbusApplication.class, args);
    }

    @Bean
    public EmbeddedServletContainerFactory createEmbeddedServletContainerFactory() {
        TomcatEmbeddedServletContainerFactory tomcatFactory = new TomcatEmbeddedServletContainerFactory();
        tomcatFactory.setPort(8080);

        return tomcatFactory;
    }

}
```

### Spring Boot 2.x.x

```java
@SpringBootApplication
public class EventbusApplication2 {

    public static void main(String[] args) {
        SpringApplication.run(EventbusApplication2.class, args);
    }

    @Bean
    public TomcatServletWebServerFactory createEmbeddedServletContainerFactory() {
        TomcatServletWebServerFactory tomcatFactory = new TomcatServletWebServerFactory();
        tomcatFactory.setPort(8086);
        return tomcatFactory;
    }

}
```