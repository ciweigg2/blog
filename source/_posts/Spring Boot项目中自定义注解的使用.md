title: Spring Boot项目中自定义注解的使用
date: 2018-07-22 05:57:33
tags: [SpringBoot]
categories: [微服务]
---
### 1.定义注解接口
```java
/**
 * @Package: com.example.config
 * @Description: 定制一个接口
 * @author: Ciwei
 * @date: 17/2/23 下午4:20
 */
@Documented
@Retention(RUNTIME)
@Target(METHOD)
public @interface MyLog {
    String value() default "日志注解";
}
```
<!--more-->
[^Documented 注解]: Documented 注解表明这个注解应该被 javadoc工具记录. 默认情况下,javadoc是不包括注解的. 但如果声明注解时指定了 @Documented,则它会被 javadoc 之类的工具处理, 所以注解类型信息也会被包括在生成的文档中
[^Inherited 注解]: 它指明被注解的类会自动继承. 更具体地说,如果定义注解时使用了 @Inherited 标记,然后用定义的注解来标注另一个父类, 父类又有一个子类(subclass),则父类的所有属性将被继承到它的子类中

* @Target(ElementType.TYPE) //接口、类、枚举、注解
* @Target(ElementType.FIELD) //字段、枚举的常量
* @Target(ElementType.METHOD) //方法
* @Target(ElementType.PARAMETER) //方法参数
* @Target(ElementType.CONSTRUCTOR) //构造函数
* @Target(ElementType.LOCAL_VARIABLE)//局部变量
* @Target(ElementType.ANNOTATION_TYPE)//注解
* @Target(ElementType.PACKAGE) ///包
* 1.RetentionPolicy.SOURCE —— 这种类型的Annotations只在源代码级别保留,编译时就会被忽略
* 2.RetentionPolicy.CLASS —— 这种类型的Annotations编译时被保留,在class文件中存在,但JVM将会忽略
* 3.RetentionPolicy.RUNTIME —— 这种类型的Annotations将被JVM保留,所以他们能在运行时被JVM或其他使用反射机制的代码所读取和使用

### 2.通过切面来实现注解
```java
/**
 * @Package: com.example.config
 * @Description: MyLog的实现类
 * @author: ciwei
 * @date: 17/2/23 下午4:22
 */
@Component
@Aspect
public class LogAspect {
    @Pointcut("@annotation(com.example.config.MyLog)")
    private void cut() { }

    /**
     * 定制一个环绕通知
     * @param joinPoint
     */
    @Around("cut()")
    public void advice(ProceedingJoinPoint joinPoint){
        System.out.println("环绕通知之开始");
        try {
            joinPoint.proceed();
        } catch (Throwable e) {
            e.printStackTrace();
        }
        System.out.println("环绕通知之结束");
    }

       //当想获得注解里面的属性，可以直接注入改注解
        @Before("cut()&&@annotation(myLog)")
            public void record(JoinPoint joinPoint, MyLog myLog) {
            System.out.println(myLog.value());
        }

    @After("recordLog()")
    public void after() {
        this.printLog("已经记录下操作日志@After 方法执行后");
    }
}
```

> 因为Aspect作用在bean上，所以先用Component把这个类添加到容器中

> @Pointcut 定义要拦截的注解

> 至于切面表达式，不需要你记住，小编我也记不住，用的时候查一下就可以了

> 切面表达式link

> @After

> @Before

> @Around

#### 2.1 获得注解中的变量
```java
      //当想获得注解里面的属性，可以直接注入改注解
        @Before("cut()&&@annotation(myLog)")
            public void record(JoinPoint joinPoint, MyLog myLog) {
            System.out.println(myLog.value());
        }
```

#### 2.2 注解中的ProceedingJoinPoint和JoinPoint说明
> AspectJ使用org.aspectj.lang.JoinPoint接口表示目标类连接点对象，如果是环绕增强时，使用org.aspectj.lang.ProceedingJoinPoint表示连接点对象，该类是JoinPoint的子接口。任何一个增强方法都可以通过将第一个入参声明为JoinPoint访问到连接点上下文的信息。我们先来了解一下这两个接口的主要方法：

**JoinPoint**
* java.lang.Object[] getArgs()：获取连接点方法运行时的入参列表；
* Signature getSignature() ：获取连接点的方法签名对象；
* java.lang.Object getTarget() ：获取连接点所在的目标对象；
* java.lang.Object getThis() ：获取代理对象本身；

**ProceedingJoinPoint**
* ProceedingJoinPoint继承JoinPoint子接口，它新增了两个用于执行连接点方法的方法：
* java.lang.Object proceed() throws java.lang.Throwable：通过反射执行目标对象的连接点处的方法；
* java.lang.Object proceed(java.lang.Object[] args) throws java.lang.Throwable：通过反射执行目标对象连接点处的方法，不过使用新的入参替换原来的入参。

### 3.演示
```java
@RestController
public class JsonRest {
    @MyLog
    @RequestMapping("/log")
    public String getLog(){
        return "<h1>Hello World</h1>";
    }
}

当访问的时候会打印出：[因为小编只用了环绕通知]
  环绕通知之开始
  环绕通知之结束
```
