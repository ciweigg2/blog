---
title: SpringBoot使用@ConstructorBinding注解进行配置属性绑定
author: Ciwei
img: ''
coverImg: ''
top: false
cover: false
toc: true
mathjax: false
password: ''
summary: ''
tags:
  - springboot
categories:
  - 综合
date: 2020-01-05 16:35:25
---

# SpringBoot使用@ConstructorBinding注解进行配置属性绑定

`SpringBoot2.2`版本发行后一些新的功能也渐渐的浮出了水面，在之前版本`SpringBoot`的配置文件与类之间的属性绑定(`@ConfigurationProperties`)是通过`Setter`方法来进行绑定对应的配置值，而从`2.2`版本开始支持了`构造函数`的方式进行绑定

<!--more-->

## **@ConstructorBinding注解**

这个注解是`SpringBoot`在2.2发行版中添加的，添加该注解的属性配置类不再需要添加`Setter`方法，不过需要添加`构造函数`，根据构造函数进行实例化属性配置类。

## **创建项目**

使用IDEA创建一个`SpringBoot`项目，在`pom.xml`中添加依赖如下所示：

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
  </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
    <exclusions>
      <exclusion>
        <groupId>org.junit.vintage</groupId>
        <artifactId>junit-vintage-engine</artifactId>
      </exclusion>
    </exclusions>
  </dependency>
</dependencies>
```

### **配置信息**

本章主要是讲解怎么把`application.yml`或者`application.properties`配置文件的内容自动映射绑定到配置类的对应属性字段上，所以我们需要在`application.yml`文件中添加部分我们自定义的配置内容，如下所示：

```yaml
# 自定义配置
minbox:
  config:
    author: Una2
    blog-address: http://blog.una2.com
```

### **配置类**

我们对应`application.yml`的配置信息，对应编写一个名为`MinBoxConfig`的映射配置类，如下所示：

```java
/**
 * 配置类
 *
 * @author 恒宇少年
 */
@ConfigurationProperties(prefix = PREFIX)
@ConstructorBinding
public class MinBoxConfig {
    /**
     * 映射绑定 "minbox.config"前缀的配置信息
     */
    public static final String PREFIX = "minbox.config";
    /**
     * 配置信息：作者
     */
    private String author;
    /**
     * 配置信息：博客地址
     */
    private String blogAddress;

    public MinBoxConfig(String author, String blogAddress) {
        this.author = author;
        this.blogAddress = blogAddress;
    }

    public String getAuthor() {
        return author;
    }

    public String getBlogAddress() {
        return blogAddress;
    }
}
```

在之前的版本我们都是使用`@Configuration`、`@ConfigurationProperties`这两个注解来进行配置映射，从`SpringBoot2.2.1.RELEASE`版本开始我们不再需要添加`@Configuration`，只要通过`@ConfigurationPropertiesScan`结合`@ConfigurationProperties`搭配使用即可，会自动扫描指定`package`下的属性配置类进行绑定。

在属性配置类上添加`@ConstructorBinding`注解，即可实现构造函数的方式进行对应字段设置值，我们只需要把绑定赋值的参数通过构造函数的方式定义。

> 在上面代码中MinBoxConfig配置类构造函数内有两个参数：author、blogAddress，所以在实例化MinBoxConfig对象时，只会从application.yml对应获取到这两个配置内容进行赋值

开启自动配置：`@ConfigurationPropertiesScan`

## **运行测试**

使用`IDEA`创建项目时会自动在`src/test/java/{packages}`创建`@SpringBootTest`注解的测试类，我们通过测试类来验证配置是否已经赋值给了配置类，如下所示：

```java
@SpringBootTest
class SpringbootConstructorBindingPropertiesApplicationTests {

    @Autowired
    private MinBoxConfig minBoxConfig;

    @Test
    void printConfig() {
        System.out.println("作者名称：" + minBoxConfig.getAuthor());
        System.out.println("作者博客地址：" + minBoxConfig.getBlogAddress());
    }

}
```

运行`printConfig()`方法后输出内容如下所示：

```bash
作者名称：Una2
作者博客地址：http://blog.una2.com
```

## **敲黑板，划重点**

`@ConfigurationProperties`这个注解可以让我们的配置文件的内容直接映射到`Java`配置类，而且通过扫描的方式自动注册到`IOC`，极大地方便了我们在项目中使用配置内容