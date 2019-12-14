cover: http://ciwei2.cn-sh2.ufileos.com/145.jpg
title: 利用Lombok编写优雅的spring依赖注入代码,去掉繁人的@Autowired
date: 2018-09-27 21:02:15
tags: [lombok,lombok实现依赖注入]
categories: [综合]
---
利用Lombok编写优雅的spring依赖注入代码,去掉繁人的@Autowired

```java
@Service
public class OrderService {
    @Autowired
    private UserService userService;

}
```

<!--more-->

下面就是spring推荐的写法：

```java
@Service
public class OrderService {
    private final UserService userService;


    @Autowired
    public OrderService(UserService userService) {
        this.userService = userService;
    }
}
```

若是注入的类太多的话呢，看起来挺繁琐的。最近偶然在网上发现使用Lombok可以写出简洁的代码:

```java
@Service
@RequiredArgsConstructor(onConstructor = @__(@Autowired))
public class OrderService {
    //这里必须是final,若不使用final,用@NonNull注解也是可以的
    private final UserService userService;

}
```

如果这个不是所有的方法都需要依赖注入 那就去掉final 加了final的都是需要依赖注入的

这样写实际上编译后和spring推荐的写法是一样的哦，是不是很简单呢