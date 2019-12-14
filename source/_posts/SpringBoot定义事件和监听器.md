cover: http://ciwei2.cn-sh2.ufileos.com/117.jpg
title: SpringBoot定义事件和监听器
date: 2018-07-23 11:21:21
tags: [SpringBoot]
categories: [微服务]
---
* 继承事件 ApplicationEvent
* 定义监听 ApplicationListener
<!--more-->
### 继承事件 ApplicationEvent
```java
public class EmailEvent extends ApplicationEvent {
    /**
     * <p>Description：</p>
     */
    private static final long serialVersionUID = 1L;
    public String address;
    public String text;

    public EmailEvent(Object source) {
        super(source);
    }

    public EmailEvent(Object source, String address, String text) {
        super(source);
        this.address = address;
        this.text = text;
    }

    public void print(){
        System.out.println("hello spring event!");
    }

}
```

### 定义监听 ApplicationListener
```java
@Component
public class EmailListener implements ApplicationListener{

    public void onApplicationEvent(ApplicationEvent event) {
        if(event instanceof EmailEvent){
            EmailEvent emailEvent = (EmailEvent)event;
            emailEvent.print();
            System.out.println("the source is:"+emailEvent.getSource());
            System.out.println("the address is:"+emailEvent.address);
            System.out.println("the email's context is:"+emailEvent.text);
        }

    }

}
```

### 代码测试
```java
  public static void main(String[] args) {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(Application.class, args);
        EmailEvent emailEvent=new EmailEvent("hello","lxchinesszz@163.com","this is a email text");
        applicationContext.publishEvent(emailEvent);
    }
```    