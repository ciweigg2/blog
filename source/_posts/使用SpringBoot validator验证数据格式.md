cover: http://ciwei2.cn-sh2.ufileos.com/139.jpg
title: 使用SpringBoot validator验证数据格式
date: 2018-08-05 16:32:02
tags: [validator]
categories: [综合]
---
SpringBoot的Web组件内部集成了hibernate-validator，所以我们这里并不需要额外的为验证再导入其他的包，接下来我们先来看看SpringBoot为我们提供了哪些验证
<!--more-->

### 内置验证
SpringBoot因为采用了hibernate-validator，所以我们直接使用hibernate-validator就可以进行数据校验，内置验证如下图2所示：
![](/images/4461954-64d1c0e22bd114a6.png)

### 使用验证
```java
public class DemoEntity implements Serializable {
    @NotBlank
    @Length(min = 2,max = 10)
    private String name;

    @Min(value = 1)
    private int age;

    @NotBlank
    @Email
    private String mail;

    @FlagValidator(values = "1,2,3")
    private String flag;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getMail() {
        return mail;
    }

    public void setMail(String mail) {
        this.mail = mail;
    }

    public String getFlag() {
        return flag;
    }

    public void setFlag(String flag) {
        this.flag = flag;
    }
}
```

我在DemoEntity创建了三个字段，分别对这三个字段进行了有效性验证。

字段name：非空校验、长度必须在2~10位之间。
字段age：最小是1岁。
字段mail：非空校验、邮箱格式

### 测试
```java
@RestController
public class IndexController {

    @Autowired
    private MessageSource messageSource;

    @RequestMapping(value = "/validator")
    public String validator(@Valid DemoEntity entity,BindingResult result) {
        if(result.hasErrors()) {
            StringBuffer msg = new StringBuffer();
            //获取错误字段集合
            List<FieldError> fieldErrors = result.getFieldErrors();
            //获取本地locale,zh_CN
            Locale currentLocale = LocaleContextHolder.getLocale();
            //遍历错误字段获取错误消息
            for (FieldError fieldError : fieldErrors) {
                //获取错误信息
                String errorMessage = messageSource.getMessage(fieldError,currentLocale);
                //添加到错误消息集合内
                msg.append(fieldError.getField()+"："+errorMessage+" , ");
            }
            return msg.toString();
        }
        return "验证通过，" + "名称：" + entity.getName()+ "年龄：" + entity.getAge() + "邮箱地址："+entity.getMail();
    }
}
```

我在控制器中注入了一个MessageSource的接口对象，这个对象是用于格式化错误消息的。根据传入的错误字段对象（FieldError）结合hibernate-validator验证的内置错误消息文件进行输出错误消息，hibernate-validator的错误消息支持国际化，所以我们获取错误消息的时候需要传入Locale对象获取本地的国际化类型

![](/images/20180805163740.png)

### 自定义验证
自定义验证需要我们提供两个文件内容，一个是注解、另外一个是对应注解继承ConstraintValidator的实现类，下面我们假如有这么个情景，我们在DemoEntity内添加一个字段flag，需要验证flag字段内容仅为1,2,3

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.PARAMETER, ElementType.FIELD})
@Constraint(validatedBy = FlagValidatorClass.class)
public @interface FlagValidator {
    //flag的有效值多个使用','隔开
    String values();
    //提示内容
    String message() default "flag不存在";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

```java
public class FlagValidatorClass implements ConstraintValidator<FlagValidator, Object> {

    //临时变量保存flag值列表
    private String values;

    //初始化values的值
    @Override
    public void initialize(FlagValidator flagValidator) {
        //将注解内配置的值赋值给临时变量
        this.values = flagValidator.values();
    }

    //实现验证
    @Override
    public boolean isValid(Object value, ConstraintValidatorContext constraintValidatorContext) {
        //分割定义的有效值
        String[] value_array = values.split(",");
        boolean isFlag = false;
        //遍历比对有效值
        for (int i =0;i<value_array.length;i++)
        {
            //存在一致跳出循环，赋值isFlag=true
            if(value_array[i].equals(value))
            {
                isFlag = true;
                break;
            }
        }
        //返回是否存在boolean
        return isFlag;
    }
}
```