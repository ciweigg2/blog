title: 【面试】spring的生命周期
date: 2019-08-22 21:18:45
tags: [java面试题]
categories: [综合]
---
### spring的生命周期

Spring作为当前Java最流行、最强大的轻量级容器框架，了解熟悉spring的生命周期非常有必要

<!--more-->

![](/images/20190822212241.png)

* 首先容器启动后，对bean进行初始化

* 按照bean的定义，注入属性

* 检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给bean，如BeanNameAware等

* 以上步骤，bean对象已正确构造，通过实现BeanPostProcessor接口，可以再进行一些自定义方法处理。如:postProcessBeforeInitialzation。

* BeanPostProcessor的前置处理完成后，可以实现postConstruct，afterPropertiesSet,init-method等方法， 增加我们自定义的逻辑，

* 通过实现BeanPostProcessor接口，进行postProcessAfterInitialzation后置处理

* 接着Bean准备好被使用啦。

* 容器关闭后，如果Bean实现了DisposableBean接口，则会回调该接口的destroy()方法

* 通过给destroy-method指定函数，就可以在bean销毁前执行指定的逻