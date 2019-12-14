title: java接口默认实现和提供给外部实现的方法
date: 2019-07-28 18:55:13
tags: [接口提供给外部实现]
categories: [综合]
---
### 介绍

很多像mybatis的插件呀 可以自己实现加载一些参数呀 那么这个要怎么做呢

<!--more-->

比如我们要用到的bean

CiweiConfiguration

```java
public interface CiweiConfiguration {

    String ciwei();

}
```

接口

```java
public interface TestService {

    String test();

}
```

接口的默认实现

```java
public class TestServiceImpl implements TestService {

    @Override
    public String test() {
        return "test1";
    }

}
```

装载接口

TestConfiguration

```java
@Configuration
public class TestConfiguration {

    @Autowired
    private TestService testService;

    @Bean
    public TestService testService(){
        return new TestServiceImpl();
    }

    @Bean
    public CiweiConfiguration ciweiConfiguration(){
        return new CiweiConfiguration() {
            @Override
            public String ciwei() {
                return testService.test();
            }
        };
    }

}
```

测试

```java
@RestController
public class TestController {

    @Autowired
    private CiweiConfiguration ciweiConfiguration;

    @RequestMapping(value = "test")
    public String test(){
        return ciweiConfiguration.ciwei();
    }

}
```

会发现测试输出使用默认的实现输出test1没什么问题

使用自定义实现覆盖默认实现

```java
@Component
public class Test2ServiceImpl implements TestService {

    @Override
    public String test() {
        return "test2";
    }

}
```

会发现测试输出使用默认的实现输出test2这样可以重新实现一些逻辑呀

当然也可以使用接口的默认实现不用去implements实现接口呀

```java
public interface TestService {

    default String test(){
        return "默认的实现";
    };

}
```

这个思想主要是用户可以实现一些自定义的加载参数等 不使用默认的方式 哈哈真厉害