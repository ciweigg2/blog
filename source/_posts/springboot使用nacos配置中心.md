cover: http://ciwei2.cn-sh2.ufileos.com/113.jpg
title: springboot使用nacos配置中心-01入门
date: 2019-01-13 13:32:24
tags: [nacos,nacos-config]
categories: [nacos]
---
### 添加maven依赖

```java
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>nacos-config-spring-boot-starter</artifactId>
            <version>0.2.1</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>nacos-config-spring-boot-actuator</artifactId>
            <version>0.2.1</version>
        </dependency>
```

<!--more-->

### application.properties

```java
nacos.config.server-addr=118.184.218.184:8848
```

### springboot启动类
```
@EnableAutoConfiguration
@SpringBootApplication
@NacosPropertySource(
		name = "custom",
		dataId = "nacos_config",
		first = true,
		groupId = "nacos_group",
		autoRefreshed = true, //自动刷新
		before = SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME,
		after = SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME
)
public class ConfigApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigApplication.class ,args);
	}

}
```

### controller测试类

```java
@RestController
public class NacosController {

    @NacosValue(value = "${name:unknown}" ,autoRefreshed = true)
    private String name;

    @Autowired
    private User user;

    @RequestMapping(value = "/sayHello")
    public String dubboSayHello(){
        System.out.println(name);
        return name;
    }

    //监听nacos配置文件的变化
    @NacosConfigListener(
            dataId = "nacos_config",
            groupId = "nacos_group",
            timeout = 500
    )
    public void onChange(String newContent) throws Exception {
        System.out.println("onChange : " + newContent);
    }

    @NacosConfigListener(
            dataId = "nacos_config2",
            groupId = "nacos_group",
            timeout = 500
    )
    public void onChange2(String newContent) throws Exception {
        System.out.println("onChange : " + newContent);
    }

}
```

### User对象

```java
@Data
@NacosConfigurationProperties(dataId = "nacos_config2" ,groupId = "nacos_group" ,autoRefreshed = true)
@Component
public class User {

    private String name;

}
```

### 配置中心添加配置

![](/images/20190113135519.png)

### 测试

> 会发现对象和变量都有值 修改配置中心的值会立马刷新变量的值

* http://localhost:8080/sayHello

![](/images/20190113142224.png)

### json配置的测试

![](/images/20190113170256.png)

```java
       try {
            String serverAddr = "118.184.218.184:8848";
            String dataId = "json";
            String group = "DEFAULT_GROUP";
            Properties properties = new Properties();
            properties.put("serverAddr", serverAddr);
            //这里设置了dev空间的配置
            properties.put("namespace" ,"7d32a129-a6ca-4ba1-a75e-96aaed89da33");
            ConfigService configService = NacosFactory.createConfigService(properties);
            // Actively get the configuration.
            String content = configService.getConfig(dataId, group, 5000);
            System.out.println(content);
        } catch (NacosException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
```

demo：https://github.com/ciweigg2/springboot-nacos-config.git