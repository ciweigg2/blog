cover: http://ciwei2.cn-sh2.ufileos.com/26.jpg
title: Dozer 轻松高效玩转DTO(Data Transfer Object)
date: 2019-06-20 21:01:45
tags: []
categories: []
---
### 介绍

对象拷贝字段不一样 对象拷贝不能拷贝List等

<!--more-->

参考：http://www.spring4all.com/article/15055

### 现状

对于分布式系统，需要在不同系统之间传递与转换域对象。因为我们不希望外部公开内部域对象，也不允许外部域对象渗入系统。传统上，数据对象之间的映射通过手工编码(getter/setter)的方式实现，或对象组装器（或转换器）来解决。我们可能会开发某种自定义映射框架来满足我们的映射转换需求，但这一切都显得不够灵巧


### Dozer

Dozer 是 Java Bean 到 Java Bean 映射器，它以递归方式将数据从一个对象复制到另一个对象

> 通常，这些 Java Bean 将具有不同的复杂类型。Dozer 支持简单属性映射，复杂类型映射，双向映射，隐式和显式映射以及递归映射。
Dozer不仅支持属性名称之间的映射，还支持在类型之间自动转换。大多数转换方案都是开箱即用的，但 Dozer 还允许您通过 XML / API 的方式指定自定义转换。

下图描绘了 Dozer 可以插入到架构中的一些常见区域。请注意，它通常用于边界（进入/退出）。 Dozer 将确保数据库中的内部域对象不会流入外部表示层或外部使用者。它还可以帮助将域对象映射到外部 API 调用，反之亦然，现在不用纠结这个图，看完下面的测试用例回看该图，柳暗花明， 文末有完整测试用例

![](/images/17985603-fbb1c5b8b6ffcbbe.gif)

### 集成 Dozer

使用 Dozer 的方式很简单，如果你使用 Maven，添加依赖到 pom.xml 中即可

```java
<dependency>
    <groupId>com.github.dozermapper</groupId>
    <artifactId>dozer-core</artifactId>
    <version>6.4.0</version>
</dependency>
```

如果你使用 Spring Boot，引入 Dozer starter 即可：

```java
<dependency>
    <groupId>com.github.dozermapper</groupId>
    <artifactId>dozer-spring-boot-starter</artifactId>
    <version>6.2.0</version>
</dependency>
```

本文主要讲述在 Spring Boot 下如何通过 Dozer 帮助我们搞定 DTO 那点事

### 使用 Dozer

默认使用

Dozer starter 默认为我们注入了 Dozer Mapper，可以直接使用，另外，文章中所有测试用例中使用 Lombok 注解简化代码

新建 StudentDomain.java 类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class StudentDomain {
    // 身份ID
    private Long id;
    // 姓名
    private String name;
    // 年龄
    private Integer age;
    // 电话
    private String mobile;
}
```

新建 StudentVo.java 类，内容同 StudentDomain.java

编写测试用例：

```java
@Autowired
private Mapper dozerMapper;

@Test
public void testDefault(){
    StudentDomain studentDomain = new StudentDomain(1024L, "tan日拱一兵", 18, "13996996996");

    StudentVo studentVo = dozerMapper.map(studentDomain, StudentVo.class);
    log.info("StudentVo: [{}]", studentVo.toString());

    studentVo.setAge(16);
    log.info("StudentDomain: [{}]", dozerMapper.map(studentVo, StudentDomain.class));
}
```

运行结果：

```java
StudentVo: [StudentVo(id=1024, name=tan日拱一兵, age=18, mobile=13996996996)]
StudentDomain: [StudentDomain(id=1024, name=tan日拱一兵, age=16, mobile=13996996996)]
```

结论：

> Dozer 默认支持同名 field 的双向映射，即隐式映射
如果仅满足这点需求，就没必要写该文章了，应用 Dozer 也为了满足我们更多定制化的需求

### 定制化使用

为满足更多的转换需求，我们需要针对 Dozer 定制化，即需要我们声明自己的 Mapper，新建 DozerConfig.java 类

```java
@Configuration
public class DozerConfig {

    @Bean
    public Mapper dozerMapper(){
        Mapper mapper = DozerBeanMapperBuilder.create()
                //指定 dozer mapping 的配置文件(放到 resources 类路径下即可)，可添加多个 xml 文件，用逗号隔开
                .withMappingFiles("dozerBeanMapping.xml")
                .withMappingBuilder(beanMappingBuilder())           
                .build();
        return mapper;
    }

    @Bean
    public BeanMappingBuilder beanMappingBuilder() {
        return new BeanMappingBuilder() {
            @Override
            protected void configure() {
                // 个性化配置添加在此
            }    
        };
    }
}
```

Dozer 完成映射有三种方式 XML, API, 注解，因官网多数都是 XML 样例，以及注解方式的局限性所在，所以本文主要使用 API 这种方式，为更好的体现 Dozer 的特性，现阶段必须以 XML API 二者结合的方式来编写测试用例，因为官网说明：

> Global config is not supported via APIMappings, API mappings are not 100% feature comparable with XML

测试用例（共 10 个）

### 用例 1

如果两个待映射的 field 不同名，Dozer 默认不会帮我们完成映射，忽略该值，所以我们需要显示映射该 field
向 StudentDomain.java 中添加学生地址信息

```java
// 地址
private String address;
```

而 StudentVo.java 中表示学生地址的信息是

```java
// 地址
private String addr;
```

我们需要在 configure 方法中显示指定映射关系

```java
@Override
protected void configure() {
    //测试所有properties，为不同名的 property 手动配置映射关系
    mapping(StudentDomain.class, StudentVo.class)
        .fields("address", "addr");
}
```

修改测试用例:

```java
@Test
public void testDifferentAddress(){
    StudentDomain studentDomain = new StudentDomain(1024L, "tan日拱一兵", 18, "13996996996", "中国");

    StudentVo studentVo = dozerMapper.map(studentDomain, StudentVo.class);
    log.info("StudentVo: [{}]", studentVo.toString());
}
```

运行结果：

> StudentVo: [StudentVo(id=1024, name=tan日拱一兵, age=18, mobile=13996996996, addr=中国)]

### 用例 2

Dozer 默认是隐式匹配，如果我们关闭隐士匹配，Dozer 只会为我们匹配我们显式指定的 field
修改 configure

```java
//关闭隐式匹配
mapping(StudentDomain.class, StudentVo.class, TypeMappingOptions.wildcard(false))
    .fields("address", "addr");
```

重新运行 用例1的测试方法 ，运行结果(只有地址做了映射)：

```java
StudentVo: [StudentVo(id=null, name=null, age=null, mobile=null, addr=中国)]
```

###用例 3

默认我们要使用 Dozer 的隐式匹配（同名字段全部匹配），但我们不想将学生的 mobile 字段做映射，我们可以通过 exclude 方法排除不想映射的字段
修改 configure

```java
//测试所有properties，为不同名的 property 手动配置映射关系，排除 mobile 字段
mapping(StudentDomain.class, StudentVo.class)
        .exclude("mobile")
```

重新运行 用例1的测试方法 ，运行结果：

```java
StudentVo: [StudentVo(id=1024, name=tan日拱一兵, age=18, mobile=null, addr=中国)]
        .fields("address", "addr");
```

### 用例 4

对象通常嵌套对象或者集合对象，Dozer 可以递归完成相关映射
将学生地址封装，同时为学生添加多门课程
新增 AddressDomain.java 和 AdressVo.java，除详细地址外所有字段相同

```java
@Data
public class AddressDomain {
    // 省
    private String province;
    // 市
    private String city;
    // 区
    private String district;
    // 详细
    private String detail;
}

@Data
public class AddressVo {
    // 省
    private String province;
    // 市
    private String city;
    // 区
    private String district;
    // 详细
    private String detailAddr;
}
```

同时创建课程类 CourseDomian.java 和 CourseVo.java, 内容相同

```java
@Data
public class CourseDomain {
    // 课程编码
    private String courseCode;
    // 课程Id
    private Integer courseId;
    // 课程名称
    private String courseName;
    // 老师名称
    private String teacherName;
}
```

同时修改 StudentDomain.java 和 StudentVo.java, 添加地址和课程集合字段：

```java
// 地址
private AddressDomain address;
// 课程集合
private List<CourseDomain> courses;
```

修改configure 配置

```java
mapping(AddressDomain.class, AddressVo.class)
    .fields("detail", "detailAddr");
```

修改测试用例：

```java
@Test
public void testCascadeObject(){
    StudentDomain studentDomain = getStudentDomain();

    StudentVo studentVo = dozerMapper.map(studentDomain, StudentVo.class);
    log.info("StudentVo: [{}]", studentVo.toString());
}
```

运行结果：

```java
StudentVo: [StudentVo(id=1024, name=t=水寒)])]
```

结论：

Dozer 会隐式递归匹配所有 field，甚至集合

###用例 5

深度匹配需求，英语老师是辅导员，需要单独匹配到 StudentVo.java 的 counsellor 字段
添加 configure mapping

```java
//测试深度索引匹配
mapping(StudentDomain.class, StudentVo.class)
        .fields("courses[0].teacherName", "counsellor");
```

重新运行测试用例，运行结果：

```java
StudentVo: [StudentVo(id=1024, name=tan日拱一兵, age=18, mobile=13996996996, address=AddressVo(province=北京, city=北京, district=海淀区, detailAddr=西二旗), courses=[CourseVo(courseCode=English, courseId=1, courseName=英语, teacherName=京晶), CourseVo(courseCode=Chinese, courseId=2, courseName=语文, teacherName=水寒)], counsellor=京晶)]
```

结论：

我们可以通过深度匹配指定字段，匹配方式以 "." 号进行分割，集合属性可以指定索引

###用例 6

修改 StudentDomain.java 的 age 字段为 Integer 类型，修改 StudentVo.java 的 age 字段为 String 类型
重新运行上述测试用例，双向映射，一切正常

结论：

Dozer 开箱即用的功能之一就是类型转换，多数类型我们不需要手动转换类型，完全交给 Dozer即可

### 用例 7

上面说到多数类型 Dozer 可以默认做转换，但是 Date 和 String 不可以，我们需要指定 date-formate 格式
为学生添加入学日期 entrollmentDate，在 StudentDomain.java 中是 String 类型，在 StudentVo.java 中是 java.util.Date 类型

```java
// 入学日期 设置为"2019-09-01 10:00:00"
private String entrollmentDate;
```

为 entrollmentDate 字段配置 date-formate ，修改 configure

```java
mapping(StudentDomain.class, StudentVo.class, TypeMappingOptions.dateFormat("yyyy-MM-dd"))
    .fields("courses[0].teacherName", "counsellor");
```

运行结果：

```java
StudentVo: [StudentVo(id=1024, name=tan日拱一兵, age=18, mobile=13996996996, address=AddressVo(province=北京, city=北京, district=海淀区, detailAddr=西二旗), courses=[CourseVo(courseCode=English, courseId=1, courseName=英语, teacherName=京晶), CourseVo(courseCode=Chinese, courseId=2, courseName=语文, teacherName=水寒)], counsellor=京晶, entrollmentDate=Sun Sep 01 00:00:00 CST 2019)]
```

我们同样可以设置全局 date-formate 和 field 级别 date-formate，全局设置，修改 dozerBeanMapping.xml, 并添加如下内容：

```java
<?xml version="1.0" encoding="UTF-8"?>
<mappings xmlns="http://dozermapper.github.io/schema/bean-mapping"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://dozermapper.github.io/schema/bean-mapping http://dozermapper.github.io/schema/bean-mapping.xsd">
    <configuration>
        <!-- 默认是 true，当发生转换错误时抛出异常，停止转换，这里设置成false，如果转换错误，继续转换 -->
        <stop-on-errors>false</stop-on-errors>
        <date-format>yyyy-MM-dd HH:mm:ss</date-format>
    </configuration>
</mappings>
```

如果同时设置了全局/类/Field 级别的 date-format，按照优先级最高的进行格式化：Field > 类 > 全局

###用例 8

我们可以为 mapping 设置 mapId， 在转换的时候指定 mapId，mapId 可以设置在类级别，也可以设置在 field 级别，实现一次定义，多处使用，同时也可以设置转换方向从默认的双向变为单向（one way）：

```java
mapping(StudentDomain.class, StudentVo.class, TypeMappingOptions.mapId("userFieldOneWay"))
.fields("age", "age", FieldsMappingOptions.useMapId("addrAllProperties"), FieldsMappingOptions.oneWay());
```

修改测试用例，指定 mapId

```java
StudentVo studentVo = dozerMapper.map(studentDomain, StudentVo.class, "userFieldOneWay");
```

###用例 9

当有些字段需要特殊处理的时候，我们需要实现自定义转换，也就是需要自定义 Converter
假设 StudentDomain.java 有 Integer 类型的 score 字段，StudentVo.java 中表示的分数则是 Enum 类型，分为 A/B/C/D 四个等级

自定义Converter，继承 DozerConverter<A, B>, 并实现其方法：

```java
public class ScoreConverter extends DozerConverter<Integer, ScoreEnum> {

    public ScoreConverter() {
        super(Integer.class, ScoreEnum.class);
    }

    @Override
    public ScoreEnum convertTo(Integer score, ScoreEnum scoreEnum) {
        if (60 <= score && score < 80){
            return ScoreEnum.C;
        }else if (80 <= score && score < 90){
            return ScoreEnum.B;
        }else if (90 <= score){
            return ScoreEnum.A;
        }else {
            return ScoreEnum.D;
        }
    }

    @Override
    public Integer convertFrom(ScoreEnum scoreEnum, Integer integer) {
        return null;
    }
}
```

修改 configure，添加 mapping：

```java
mapping(StudentDomain.class, StudentVo.class)
    .fields("score", "score", customConverter(ScoreConverter.class));
```

运行结果：

```java
StudentVo: [StudentVo(id=1024, name=tan日拱一兵, age=18, mobile=13996996996, address=AddressVo(province=北京, city=北京, district=海淀区, detailAddr=西二旗), courses=[CourseVo(courseCode=English, courseId=1, courseName=英语, teacherName=京晶), CourseVo(courseCode=Chinese, courseId=2, courseName=语文, teacherName=水寒)], counsellor=null, entrollmentDate=Sun Sep 01 10:00:00 CST 2019, score=A)]
```

###用例 10

Dozer 可以通过实现 DozerEventListener 接口实现 mapping 的事件监听，在 mapping 的时候做全局业务：

```java
@Slf4j
public class StudentListener implements DozerEventListener {

    @Override
    public void mappingStarted(DozerEvent dozerEvent) {
        log.info("mappingStarted");
    }

    @Override
    public void preWritingDestinationValue(DozerEvent dozerEvent) {
        log.info("preWritingDestinationValue");

    }

    @Override
    public void postWritingDestinationValue(DozerEvent dozerEvent) {
        log.info("postWritingDestinationValue");
    }

    @Override
    public void mappingFinished(DozerEvent dozerEvent) {
        log.info("mappingFinished");
    }
}
```

Dozer 可使用的样例远不止这些，这里罗列出我们最常见的一些业务问题；看完这些用例之后，请回看文章开头的图，我们可以在系统边界处充分利用 Dozer，满足 DTO 的一切需求

### Dozer 支持的数据类型转换（双向）

在此列举出 Dozer 支持的数据类型转换（双向）

* Primitive to Primitive Wrapper
* Primitive to Custom Wrapper
* Primitive Wrapper to Primitive Wrapper
* Primitive to Primitive
* Complex Type to Complex Type
* String to Primitive
* String to Primitive Wrapper
* String to Complex Type if the Complex Type contains a String constructor
* String to Map
* Collection to Collection
* Collection to Array
* Map to Complex Type
* Map to Custom Map Type
* Enum to Enum
* Each of these can be mapped to one another: java.util.Date, java.sql.Date, java.sql.Time, java.sql.Timestamp, java.util.Calendar, java.util.GregorianCalendar
* String to any of the supported Date/Calendar Objects.
* Objects containing a toString() method that produces a long representing time in (ms) to any supported Date/Calendar object.

同时看官网 Release 版本，Dozer 现已支持 proto 类型的转换的支持，即支持 gRPC；

### 总结

Dozer 可以高效的处理我们日常 DTO 业务，一次 mapping 定义，多处使用, Dozer 与 Lombok 结合使用极大的简化了我们的代码编写量，代码更加工整简洁。同时 [Dozer Github](https://github.com/DozerMapper/dozer) 也保持活跃更新，可以追踪更多新特性，本文 demo 地址：Dozer Demo Github。如果你在业务中需要一些特殊的转换规则，欢迎留言交流，我们一起探讨实现b.com/DozerMapper/dozer) 也保持活跃更新，可以追踪更多新特性，本文 demo 地址：[]()Dozer Demo Github如果你在业务中需要一些特殊的转换规则，欢迎留言交流，我们一起探讨实现b.com/DozerMapper/dozer) 也保持活跃更新，可以追踪更多新特b.com/DozerMapper/dozer) 也保持活跃更新，可以追踪更多新特性，本文 demo 地址：[Dozer Demo Github](https://github.com/ciweigg2/springboot-doze-examples)如果你在业务中需要一些特殊的转换规则，欢迎留言交流，我们一起探讨实现