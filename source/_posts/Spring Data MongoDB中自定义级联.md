cover: http://ciwei2.cn-sh2.ufileos.com/102.jpg
title: Spring Data MongoDB中自定义级联
date: 2018-07-30 21:27:57
tags: [mongodb]
categories: [综合]
---
Spring Data MongoDB中自定义级联
原文链接：http://www.baeldung.com/cascading-with-dbref-and-lifecycle-events-in-spring-data-mongodb
<!--more-->

### 1. 概述
本教程将继续探索Spring Data MongoDB的一些核心特性——@DBRef 注解和life-cycle事件。

### 2. @DBRef
映射框架不支持在其他document中存储parent-child关系和嵌入式document。我们可以做的是——我们可以分别存储它们，并使用DBRef来引用这些document。

当对象从MongoDB加载时，这些引用将会被立即解析，我们将返回一个映射的对象，它看起来与存储在我们的master document中一样。

让我们看一些代码：
```java
@DBRef
private EmailAddress emailAddress;
```

EmailAddress的定义如下所示:

```java
@Document
public class EmailAddress {
    @Id
    private String id;

    private String value;

    // standard getters and setters
}
```

注意，映射框架不处理级联操作。举个例子，如果我们触发了一个parent document的save，child document就不会自动保存——如果我们想要保存的话，我们需要显式地触发这个child document的保存。

这正是life cycle事件的用武之地。

### 3. Lifecycle 事件
Spring Data MongoDB发布了一些非常有用的life cycle事件——比如onBeforeConvert、onBeforeSave、onAfterSave、onAfterLoad和onAfterConvert。

为了拦截其中一个事件，我们需要注册AbstractMappingEventListener的子类，并覆盖其中的一个方法。当事件被触发时，我们的监听器将被调用，领域对象被传入。

#### 3.1. 级联保存的简单实现

让我们看一下前面的示例——用emailAddress保存user。我们现在可以监听onBeforeConvert事件，它将在领域对象进入转换器之前被调用：

```java
public class UserCascadeSaveMongoEventListener extends AbstractMongoEventListener<Object> {
    @Autowired
    private MongoOperations mongoOperations;

    @Override
    public void onBeforeConvert(BeforeConvertEvent<Object> event) { 
        Object source = event.getSource(); 
        if ((source instanceof User) && (((User) source).getEmailAddress() != null)) { 
            mongoOperations.save(((User) source).getEmailAddress());
        }
    }
}    
```

现在我们只需要将监听器注册到MongoConfig：

```java
@Bean
public UserCascadeSaveMongoEventListener userCascadingMongoEventListener() {
    return new UserCascadeSaveMongoEventListener();
}
```

Or 使用XML形式:

```java
<bean class="org.baeldung.event.UserCascadeSaveMongoEventListener" />
```

我们已经完成了级联语义——尽管只是针对user。

#### 3.2. 一个通用的级联实现
现在让我们通过使级联功能通用来改进以前的解决方案。让我们从定义一个自定义注解开始：
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface CascadeSave {
    //
}
```
现在让我们来处理我们的自定义监听器，以通用的方式处理这些字段，而不需要将它们转换为任何特定的entity：
```java
public class CascadeSaveMongoEventListener extends AbstractMongoEventListener<Object> {

    @Autowired
    private MongoOperations mongoOperations;

    @Override
    public void onBeforeConvert(BeforeConvertEvent<Object> event) { 
        Object source = event.getSource(); 
        ReflectionUtils.doWithFields(source.getClass(), 
          new CascadeCallback(source, mongoOperations));
    }
}
```
我们使用的是Spring的反射工具类，在满足我们标准的所有字段上执行回调：
```java
@Override
public void doWith(Field field) throws IllegalArgumentException, IllegalAccessException {
    ReflectionUtils.makeAccessible(field);

    if (field.isAnnotationPresent(DBRef.class) && 
      field.isAnnotationPresent(CascadeSave.class)) {

        Object fieldValue = field.get(getSource());
        if (fieldValue != null) {
            FieldCallback callback = new FieldCallback();
            ReflectionUtils.doWithFields(fieldValue.getClass(), callback);

            getMongoOperations().save(fieldValue);
        }
    }
}
```

正如您所看到的，我们正在查找同时使用了 DBRef 和 CascadeSave 注解的字段。只要找到了符合条件的字段，我们就保存child entity。

让我们看一下FieldCallback类，我们用它来检查child entity是否有@Id注解：

```java
public class FieldCallback implements ReflectionUtils.FieldCallback {
    private boolean idFound;

    public void doWith(Field field) throws IllegalArgumentException, IllegalAccessException {
        ReflectionUtils.makeAccessible(field);

        if (field.isAnnotationPresent(Id.class)) {
            idFound = true;
        }
    }

    public boolean isIdFound() {
        return idFound;
    }
}
```

最后，为了让所有的级联操作都保持相同的代码风格，当然,我们需要给emailAddress 字段也加上@CascadeSave注解：

```java
@DBRef
@CascadeSave
private EmailAddress emailAddress;
```

#### 3.3. 级联测试
现在让我们看一个场景——我们用emailAddress保存User，并且保存操作会自动地级联这个嵌入的entity：

```java
User user = new User();
user.setName("Brendan");
EmailAddress emailAddress = new EmailAddress();
emailAddress.setValue("b@gmail.com");
user.setEmailAddress(emailAddress);
mongoTemplate.insert(user);
```

让我们核对下数据库:

```java
{
    "_id" : ObjectId("55cee9cc0badb9271768c8b9"),
    "name" : "Brendan",
    "age" : null,
    "email" : {
        "value" : "b@gmail.com"
    }
}
```

### 4. 总结
在本文中，我们展示了Spring Data MongoDB的一些很酷的特性——@DBRef注解、life cycle事件以及我们如何智能地处理级联。
示例：[mongodb](https://github.com/eugenp/tutorials/tree/master/spring-data-mongodb)

如果我们触发了一个parent document的save，child document就不会自动保存——如果我们想要保存的话，我们需要显式地触发这个child document的保存