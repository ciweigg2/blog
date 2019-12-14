cover: http://ciwei2.cn-sh2.ufileos.com/119.jpg
title: SpringBoot结合Nepxion EventBus异步操作
date: 2018-08-11 18:06:28
tags: [EventBus异步]
categories: [综合]
---
Nepxion EventBus是一款基于Google Guava通用事件派发机制的事件总线组件，它采用Spring framework AOP机制，提供注解调用方式，支持异步和同步两种方式
### 简介
* 实现基于@EventBus注解开启EventBus机制
* 实现异步模式下(默认)，子线程中收到派发的事件，基于@EventBus(async = false)，来切换是同步还是异步
* 实现批量派发事件
* 实现同步模式下，主线程中收到派发的事件
* 实现线程隔离技术，并定制化配置线程池
* 实现事件对象的多元化，可以发布和订阅Java基本类型，也可以利用框架内置的Event类型，当然也可以使用任意自定义类型
<!--more-->

### 最新版本兼容
* Spring 4.x.x和Spring Boot 1.x.x
* Spring 5.x.x和Spring Boot 2.x.x

### 依赖
可以前往https://github.com/Nepxion/EventBus查看最新版本
```java
        <dependency>
            <groupId>com.nepxion</groupId>
            <artifactId>eventbus</artifactId>
            <version>2.0.2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>

        <dependency>
            <groupId>com.nepxion</groupId>
            <artifactId>eventbus-aop-starter</artifactId>
            <version>2.0.2</version>
        </dependency>
```

### 策略
* EventBus事件控制器（Controller）策略
* 可以由单个Controller控制缺省identifier的EventBus事件（在Google Guava内部定义缺省identifier的值为'default'）。用法如下：

```java
事件发布端：
eventControllerFactory.getAsyncController(identifier).post("abc"); // 异步发送
eventControllerFactory.getSyncController(identifier).post("abc"); // 同步发送
```

```java
事件订阅端：
@EventBus(identifier = "xyz") // 订阅异步消息，async不指定，默认为true
public class MySubscriber {
}
@EventBus(identifier = "xyz", async = false) // 订阅同步消息
public class MySubscriber {
}
```

> 注意：事件发布端和订阅端的identifier一定要一致

* EventBus线程池（ThreadPool）策略
* 配置如下： 线程池配置，参考application.properties，可以不需要配置，那么采取如下默认值

```java
# Thread Pool Config
# Multi thread pool，是否线程隔离。如果是，那么每个不同identifier的事件都会占用一个单独线程池，否则共享一个线程池
threadPoolMultiMode=false
# 共享线程池的名称
threadPoolSharedName=EventBus
# 是否显示自定义的线程池名
threadPoolNameCustomized=true
# CPU unit（CPU核数单位，例如在8核心CPU上，threadPoolCorePoolSize配置为2，那么最终核心线程数为16，下同）
threadPoolCorePoolSize=1
# CPU unit
threadPoolMaximumPoolSize=2
threadPoolKeepAliveTime=900000
threadPoolAllowCoreThreadTimeout=false
# LinkedBlockingQueue, ArrayBlockingQueue, SynchronousQueue
threadPoolQueue=LinkedBlockingQueue
# CPU unit (Used for LinkedBlockingQueue or ArrayBlockingQueue)
threadPoolQueueCapacity=128
# BlockingPolicyWithReport, CallerRunsPolicyWithReport, AbortPolicyWithReport, RejectedPolicyWithReport, DiscardedPolicyWithReport
threadPoolRejectedPolicy=BlockingPolicyWithReport
```

### 示例
调用入口1，异步模式(默认)下接收事件
```java
package com.nepxion.eventbus.example.service;

/**
 * <p>Title: Nepxion EventBus</p>
 * <p>Description: Nepxion EventBus AOP</p>
 * <p>Copyright: Copyright (c) 2017-2050</p>
 * <p>Company: Nepxion</p>
 * @author Haojun Ren
 * @version 1.0
 */

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

import com.google.common.eventbus.Subscribe;
import com.nepxion.eventbus.annotation.EventBus;
import com.nepxion.eventbus.core.Event;

@EventBus
@Service
public class MySubscriber1 {
    private static final Logger LOG = LoggerFactory.getLogger(MySubscriber1.class);

    @Subscribe
    public void subscribe(String event) {
        LOG.info("子线程接收异步事件 - {}，String类型", event);
    }

    @Subscribe
    public void subscribe(Long event) {
        LOG.info("子线程接收异步事件 - {}，Long类型", event);
    }

    @Subscribe
    public void subscribe(Boolean event) {
        LOG.info("子线程接收异步事件 - {}，Boolean类型", event);
    }

    @Subscribe
    public void subscribe(Event event) {
        LOG.info("子线程接收异步事件 - {}，内置类型Event", event);
    }
}
```

调用入口2，同步模式下接收事件

```java
package com.nepxion.eventbus.example.service;

/**
 * <p>Title: Nepxion EventBus</p>
 * <p>Description: Nepxion EventBus AOP</p>
 * <p>Copyright: Copyright (c) 2017-2050</p>
 * <p>Company: Nepxion</p>
 * @author Haojun Ren
 * @version 1.0
 */

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

import com.google.common.eventbus.Subscribe;
import com.nepxion.eventbus.annotation.EventBus;
import com.nepxion.eventbus.core.Event;

@EventBus(async = false)
@Service
public class MySubscriber2 {
    private static final Logger LOG = LoggerFactory.getLogger(MySubscriber2.class);

    @Subscribe
    public void subscribe(String event) {
        LOG.info("主线程接收同步事件 - {}，String类型", event);
    }

    @Subscribe
    public void subscribe(Long event) {
        LOG.info("主线程接收同步事件 - {}，Long类型", event);
    }

    @Subscribe
    public void subscribe(Boolean event) {
        LOG.info("主线程接收同步事件 - {}，Boolean类型", event);
    }

    @Subscribe
    public void subscribe(Event event) {
        LOG.info("主线程接收同步事件 - {}，内置类型Event", event);
    }
}
```

调用入口3，派发事件

```java
package com.nepxion.eventbus.example.service;

/**
 * <p>Title: Nepxion EventBus</p>
 * <p>Description: Nepxion EventBus AOP</p>
 * <p>Copyright: Copyright (c) 2017-2050</p>
 * <p>Company: Nepxion</p>
 * @author Haojun Ren
 * @version 1.0
 */

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.nepxion.eventbus.core.Event;
import com.nepxion.eventbus.core.EventControllerFactory;

@Service
public class MyPublisher {
    private static final Logger LOG = LoggerFactory.getLogger(MyPublisher.class);

    @Autowired
    private EventControllerFactory eventControllerFactory;

    public void publish() {
        LOG.info("发送事件...");

        // 异步模式下(默认)，子线程中收到派发的事件
        eventControllerFactory.getAsyncController().post("Sync Event String Format");

        // 同步模式下，主线程中收到派发的事件
        // 事件派发接口中eventControllerFactory.getSyncController(identifier)必须和@EnableEventBus参数保持一致，否则会收不到事件
        eventControllerFactory.getSyncController().post("Sync Event String Format");

        // 异步模式下(默认)，子线程中收到派发的事件
        eventControllerFactory.getAsyncController().post(12345L);

        // 同步模式下，主线程中收到派发的事件
        // 事件派发接口中eventControllerFactory.getSyncController(identifier)必须和@EnableEventBus参数保持一致，否则会收不到事件
        eventControllerFactory.getSyncController().post(Boolean.TRUE);

        // 异步模式下(默认)，子线程中收到派发的事件
        eventControllerFactory.getAsyncController().postEvent(new Event("Async Event"));

        // 同步模式下，主线程中收到派发的事件
        // 事件派发接口中eventControllerFactory.getSyncController(identifier)必须和@EnableEventBus参数保持一致，否则会收不到事件
        eventControllerFactory.getSyncController().postEvent(new Event("Sync Event"));
    }
}
```

自己自定义的对象测试

```java
package com.example.eventbus.model;

import lombok.Data;

@Data
public class User {

    private String name;

    private String message;

}
```

```java
package com.example.eventbus.service;

import com.example.eventbus.model.User;
import com.nepxion.eventbus.core.EventControllerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * 自定义事件 测试自定义对象
 */
@Service
public class MyEventBusPublisher {

    @Autowired
    private EventControllerFactory eventControllerFactory;

    public void publish() {
        User user = new User();
        user.setMessage("测试eventBus");
        user.setName("测试");
        eventControllerFactory.getAsyncController().post(user);
    }

}
```

```java
package com.example.eventbus.service;

import com.alibaba.fastjson.JSONObject;
import com.example.eventbus.model.User;
import com.google.common.eventbus.Subscribe;
import com.nepxion.eventbus.annotation.EventBus;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

/**
 * 接收事件
 */
@EventBus
@Service
@Slf4j
public class MyEventBusSubscriber {

    @Subscribe
    public void subscribe(User user) {
        log.info("子线程接收异步事件 - {}，User类型", JSONObject.toJSONString(user));
    }

}
```

测试入口

```java
package com.example.eventbus.controller;

import com.example.eventbus.service.MyEventBusPublisher;
import com.example.eventbus.service.MyPublisher;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

/**
 * 测试事件
 */
@RestController
public class ResourceController {

  @Autowired
  private MyPublisher myPublisher;

  @Autowired
  private MyEventBusPublisher myEventBusPublisher;

  @RequestMapping(value = "/eventBus")
  public void eventBus(){
    myPublisher.publish();
    myEventBusPublisher.publish();
  }

}
```

测试结果 对应的类型 和请求方式会找到对应的方法去监听执行的
```java
2018-08-11 18:32:16.184  INFO 1608 --- [nio-8080-exec-5] c.example.eventbus.service.MyPublisher   : 发送事件...
2018-08-11 18:32:16.184  INFO 1608 --- [nio-8080-exec-5] c.e.eventbus.service.MySubscriber2       : 主线程接收同步事件 - Sync Event String Format，String类型
2018-08-11 18:32:16.184  INFO 1608 --- [.0.105-thread-5] c.e.eventbus.service.MySubscriber1       : 子线程接收异步事件 - Sync Event String Format，String类型
2018-08-11 18:32:16.184  INFO 1608 --- [nio-8080-exec-5] c.e.eventbus.service.MySubscriber2       : 主线程接收同步事件 - true，Boolean类型
2018-08-11 18:32:16.184  INFO 1608 --- [nio-8080-exec-5] c.e.eventbus.service.MySubscriber2       : 主线程接收同步事件 - com.nepxion.eventbus.core.Event@b1eda58[
  source=Sync Event
]，内置类型Event
2018-08-11 18:32:16.184  INFO 1608 --- [.0.105-thread-6] c.e.eventbus.service.MySubscriber1       : 子线程接收异步事件 - 12345，Long类型
2018-08-11 18:32:16.184  INFO 1608 --- [.0.105-thread-7] c.e.eventbus.service.MySubscriber1       : 子线程接收异步事件 - com.nepxion.eventbus.core.Event@6b019f65[
  source=Async Event
]，内置类型Event
2018-08-11 18:32:16.185  INFO 1608 --- [.0.105-thread-6] c.e.e.service.MyEventBusSubscriber       : 子线程接收异步事件 - {"message":"测试eventBus","name":"测试"}，User类型
```

### 扩展自定义identifier

```java
package com.example.eventbus.controller;

import com.example.eventbus.service.identifier.IdentifierPublisher;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 指定identifier测试 只会接收identifier标识匹配的事件
 */
@RestController
public class IdentifierController {

    @Autowired
    private IdentifierPublisher identifierPublisher;

    @RequestMapping(value = "/identifier")
    public void eventBus(){
        identifierPublisher.publish();
    }

}
```

```java
package com.example.eventbus.service.identifier;

import com.example.eventbus.model.User;
import com.nepxion.eventbus.core.EventControllerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * 推送自定义identifier的事件
 */
@Service
public class IdentifierPublisher {

    @Autowired
    private EventControllerFactory eventControllerFactory;

    public void publish() {
        User user = new User();
        user.setMessage("测试identifier-eventBus");
        user.setName("identifier");
        eventControllerFactory.getAsyncController("ciwei").post("测试identifier指定的接收参数");
    }

}
```

```java
package com.example.eventbus.service.identifier;

import com.google.common.eventbus.Subscribe;
import com.nepxion.eventbus.annotation.EventBus;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

/**
 * 接收identifier相同的事件
 */
@EventBus(identifier = "ciwei")
@Service
@Slf4j
public class IdentifierSubscriber {

    @Subscribe
    public void subscribe(String identifier) {
        log.info("子线程接收指定identifier异步事件 - {}，identifier-ciwei", identifier);
    }

}
```

输出结果如下 只有指定了identifier相同的方法才会监听到

```java
2018-08-11 18:34:17.052  INFO 1608 --- [.0.105-thread-2] c.e.e.s.identifier.IdentifierSubscriber  : 子线程接收指定identifier异步事件 - 测试identifier指定的接收参数，identifier-ciwei
```




