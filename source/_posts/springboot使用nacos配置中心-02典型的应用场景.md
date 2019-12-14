cover: http://ciwei2.cn-sh2.ufileos.com/112.jpg
title: springboot使用nacos配置中心-02典型的应用场景
date: 2019-01-13 14:23:57
tags: [nacos,nacos-config]
categories: [nacos]
---
### 数据库连接信息
曾经有朋友跟我聊过一个问题，“业务飞速发展，团队越来越大，人员流动也相对频繁起来，怎么才能更好的保证数据的安全性，不被泄露呢？”。他提到这样一个场景，公司创立初期，服务后端的代码都是他一行一行码出来的，当时只有他一个人，后端与数据库的连接配置信息也就直接放置在项目的配置文件中。他使用的是 Spring Boot 框架，配置信息就是存放在 application.properties 中，使用 Spring 的 profile 属性保证不同环境连接不同的数据库。如下所示：

<!--more-->

生产环境：application-prod.properties
```java
spring.datasource.url=生产环境的数据库连接地址
spring.datasource.username=生产环境的数据库用户账号
spring.datasource.password=生产环境的数据库用户密码
```

开发环境：application-dev.properties
```java
spring.datasource.url=开发环境的数据库连接地址
spring.datasource.username=开发环境的数据库用户账号
spring.datasource.password=开发环境的数据库用户密码
```
测试、预发环境也是类似。这种将数据库连接信息直接放置在配置文件中，跟着项目代码一起通过 Git 管理，的确是有蛮大的数据泄露的风险。试想，一个新来不久的小伙伴，他一当要投入研发工作，有 Git Pull 代码的权限之后，代表他可能就拥有了直接操作线上数据库的权限了。当时的我给他建议可以通过以下几个方面去降低数据风险：

将数据库连接信息等敏感配置从项目中剥离；
数据库增加 IP 白名单连接限制；
最小权限原则：每个账号只配置所必需的权限，避免删表删库等高危操作；
定期修改数据库账号、密码。
回想起来，我当时给的建议并没有完全解决他的问题，甚至还带来了其他一些问题。例如，上述的第一点，“将敏感配置从项目剥离”，剥离出来的敏感配置存放到哪里？怎么管理这些配置呢？也许，你会想到，存放到应用部署机器的环境变量或某个文件中。不，一样有风险，说不定哪天开发同学必须得登录上机器排查问题，就有泄露的风险，总之，得尽可能地做到在整个开发流程都不会有任何泄露的风险。应用中可能不只是连接一个数据源，分库分表的情况，不同数据存储（如 MySQL / Redis / Elasticsearch 等）的情况，还有，其他更多敏感配置项，配置数据的增多会给管理带来不便。

另外，“定期修改数据库账号、密码”，修改后我能怎么方便快捷的下发到所有应用程序中呢？既然是敏感配置，其变更也会带来不少的风险，我需要能先到小量的几台机器验证，保证对业务无影响，我再全部下发到其他所有的机器上去，是否还得有“灰度发布”的功能呢？账号密码修改下发后，应用出现异常，影响到业务了，我要怎么快速地回滚呢？是否还得有“版本控制”、“快速回滚”的功能呢？不是所有的开发同学都有权限能修改敏感配置信息，是否还需要有“权限管控”的功能？对敏感配置的任何操作都应该被记录，是否还需要有“变更审计”的功能呢？

现在，我有了更好的建议：使用 Nacos 配置管理模块，将敏感配置信息都存放到 Nacos 中。Nacos 配置管理，其中一个立身之本就是为敏感配置保驾护航。它提供上述场景所需的功能，通过命名空间区分不同环境（开发、测试、预发、生产），通过“版本控制”保证变更可追溯，通过“快速回滚”保证错误变更时影响最小，通过的“灰度发布”功能保障配置安全平稳地变更，还有更多更全面功能（权限管控、变更审计等）即将支持。

那么，怎么将敏感配置项目的配置文件中迁移到 Nacos 中呢？下面以 Spring Boot 连接 MySQL 为例：

添加依赖
```java
<dependency>
    <groupId>com.alibaba.boot</groupId>
    <artifactId>nacos-config-spring-boot-starter</artifactId>
    <version>0.2.1</version>
</dependency>
```
注意 Spring Boot 1.x 使用 nacos-config-spring-boot-starter 0.1.x 版本，Spring Boot 2.x 使用 nacos-config-spring-boot-starter 0.2.x 版本。

在 application.properties 中添加 Nacos 连接配置
```java
nacos.config.server-addr=127.0.0.1:8848
```
这里是简单的示例，在实际生产中，还需配置 Nacos 命名空间信息（区分环境）、鉴权信息（如 AccessKey、SecretKey 等，即将支持的权限访问控制）。而 Nacos 配置模块对应的阿里云产品 ACM，借助于 ECS 实例 RAM 角色，最终能到达连 AccessKey、SecretKey 都不需要填写的目的。

添加 @NacosPropertySource 注解
```java
@SpringBootApplication
@NacosPropertySource(dataId = "mysql.properties")
public class SpringBootMySQLApplication {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
在本地启动的 Nacos 控制台上新增 dataId 为 mysql.properties 的配置，配置内容为 MySQL 连接配置信息：

```java
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=
```

![](/images/20190113144811.png)

通过这四个简单的步骤，就将 MySQL 连接信息从原来的 application.properties 迁移到 Nacos 的，让 Nacos 将敏感配置管控起来，大大降低数据泄露的风险。同时，Nacos 配置管理提供的“统一管控”、“版本控制”、“快速回滚”等强大的功能也为其运维管理带来极大的便利。

### 可以使用命名空间来区分环境打包上线

```java
nacos.config.namespace=7d32a129-a6ca-4ba1-a75e-96aaed89da33
```

![](/images/20190113142814.png)

demo:https://github.com/ciweigg2/nacos-spring-boot-config-mysql.git