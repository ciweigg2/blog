title: springboot2.2.0新特性@ConstructorBinding
date: 2019-11-01 14:09:00
tags: [springboot]
categories: [综合]
---
### 构造函数绑定、

使用final修饰使其不能改变

不可变的 @ConfigurationProperties 绑定

配置属性现在支持基于构造函数的绑定，该绑定使 @ConfigurationProperties 注释的类不可变。可以通过使用 @ConstructorBinding 注释 @ConfigurationProperties 类或其构造函数之一来启用基于构造函数的绑定。可以在配置属性绑定提供的构造函数参数上使用 @DefaultValue 和 @DateTimeFormat 之类的注释

<!--more-->

```java
@ConfigurationProperties("acme")
@Data
@ConstructorBinding
public class AcmeProperties {

    private final boolean enabled;

    private final InetAddress remoteAddress;

    private final Security security;

    public AcmeProperties(boolean enabled, InetAddress remoteAddress, Security security) {
        this.enabled = enabled;
        this.remoteAddress = remoteAddress;
        this.security = security;
    }

    public static class Security {

        private final String username;

        private final String password;

        private final List<String> roles;

        public Security(String username, String password,
                        @DefaultValue("USER") List<String> roles) {
            this.username = username;
            this.password = password;
            this.roles = roles;
        }

    }

}
```

```java
@Configuration
@EnableConfigurationProperties(value = AcmeProperties.class)
public class AcmeConfiguartion {

    @Autowired
    private AcmeProperties acmeProperties;

}
```

如果您的类具有多个构造函数，则还可以@ConstructorBinding直接在应绑定的构造函数上使用

参考：https://docs.spring.io/spring-boot/docs/2.2.0.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-constructor-binding