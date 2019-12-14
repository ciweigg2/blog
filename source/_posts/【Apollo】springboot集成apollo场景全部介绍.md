title: 【Apollo】springboot集成apollo场景全部介绍
date: 2019-08-17 16:46:48
tags: [springboot,apollo]
categories: [综合]
---
### 集成

添加maven依赖

```xml
<dependency>
    <groupId>com.ctrip.framework.apollo</groupId>
    <artifactId>apollo-client</artifactId>
    <version>1.4.0</version>
</dependency>
```

<!--more-->

### 使用场景

apollo在springboot中的使用场景介绍 这个必须手动刷新 配置才会更新

#### @ConfigurationProperties

使用@ConfigurationProperties注解配合自动加载配置

```java
/**
 * 测试apollo自动装配 使用@ConfigurationProperties读取的配置 需要手动刷新
 */
@ConfigurationProperties(prefix = "i.love")
@Data
@RefreshScope
@Configuration
public class Love {

  private String u;

}
```

新建监听类

```java
@Configuration
@Slf4j
public class LoveConfig {

   @Autowired
   ApplicationContext context;

   @Autowired
   private RefreshScope refreshScope;

   @Autowired
   private Love love;

   /**
    * @ApolloConfigChanageListener 会监听到i.love前缀值的变化后重新为love的属性赋值   更新@RefreshScope的注解的bean
    *
    * @param changeEvent
    */
   @ApolloConfigChangeListener(interestedKeyPrefixes = "i.love")
   private void onChange(ConfigChangeEvent changeEvent) {
	   refreshScope.refresh("love");
	   log.info("刷新bean成功{}" , JSONObject.toJSON(love));
   }

}
```

#### springboot自带的条件注解使用

在配置中心添加dynamic.enable为true的配置 springboot程序启动会默认加载 条件注解很多就举例一个吧满足dynamic.enable=true才会加载bean

```java
@Bean
@ConditionalOnProperty(name = "dynamic.enable" ,havingValue = "true")
```

#### @ApolloJsonValue

这个注解主要读取key的值为json格式的数据呀 下面读取配置中心key为new-db值为json格式的数据

```java
@ApolloJsonValue("${new-db}")
private DynamicDataSourceProperties dynamicDataSourceProperties;
```

也可以读取list的呀

```java
@ApolloJsonValue("${new-db:[]}")
private List<DynamicDataSourceProperties> dynamicDataSourceProperties;
```

#### 获取非properties格式namespace的配置

yaml/yml格式的namespace

apollo-client 1.3.0版本开始对yaml/yml做了更好的支持，使用起来和properties格式一致

```java
Config config = ConfigService.getConfig("application.yml");
String someKey = "someKeyFromYmlNamespace";
String someDefaultValue = "someDefaultValueForTheKey";
String value = config.getProperty(someKey, someDefaultValue);
```

非yaml/yml格式的namespace

获取时需要使用ConfigService.getConfigFile接口并指定Format，如ConfigFileFormat.XML

```java
String someNamespace = "test";
ConfigFile configFile = ConfigService.getConfigFile("test", ConfigFileFormat.XML);
String content = configFile.getContent();
```

### 总结

暂时就发现了这几个比较好用的 如果有更好的 希望告诉我呀

demo：https://github.com/ciweigg2/apollo-dynamic-datasource