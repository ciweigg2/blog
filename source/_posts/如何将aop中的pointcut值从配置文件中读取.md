title: 如何将aop中的pointcut值从配置文件中读取
date: 2019-12-02 17:53:47
tags: [aop]
categories: [综合]
---
### 背景

改造老项目，须要加一个aop来拦截所的web Controller请求做一些处理，由于老项目比较多，且包的命名也不统一，又不想每个项目都copy一份相同的代码，这样会导致后以后升级很麻烦，不利于维护。于是我们想做成一个统一的jar包来给各项目引用，这样每个项目只须要引用该jar，然后配置对应的切面值就可以了。

<!--more-->

我们都知道，java中的注解里面的值都是一个常量， 如：

```java
@Pointcut("execution(* com.demo.Serviceable+.*(..))")
```

这种方式原则上是没有办法可以进行改变的。但是我们又要实现这将aop中的切面值做成一个动态配置的，每个项目的值的都不一样的，该怎么办呢？

首先，我们可以先创建一个类来实现 MethodInterceptor 类 ：

```java
class LogAdvice implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("Before method");//这里做你的before操作
        Object result = invocation.proceed();
        System.out.println("After method");//这里做你的after操作
        return result;
    }
}
```

然后创建一个Configuration类，创建Bean：

```java
@Configuration
public class ConfigurableAdvisorConfig {

    @Value("${pointcut.property}")
    private String pointcut;

    @Bean
    public AspectJExpressionPointcutAdvisor configurabledvisor() {
        AspectJExpressionPointcutAdvisor advisor = new AspectJExpressionPointcutAdvisor();
        advisor.setExpression(pointcut);
        advisor.setAdvice(new LogAdvice ());
        return advisor;
    }
}
```

这里面的 pointcut.property值来自于你的application.properties 等配置文件。

这样，各项目只须要引用该jar，然后在配置文件中指定要拦截的pointcut就可以了。