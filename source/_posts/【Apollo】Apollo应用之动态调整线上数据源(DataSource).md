title: 【Apollo】Apollo应用之动态调整线上数据源(DataSource)
date: 2019-08-11 10:37:45
tags: [apollo]
categories: [综合]
---
### 实现思路

参考：http://www.kailing.pub/article/index/arcid/198.html

通过对主流数据源（c3p0,dbcp2,tomcat jdbc,Hikari）实现的代理，来动态管理应用到数据库的连接，以及实现应用端的读写分离数据链接策略。如上，spring已经有个抽象类AbstractRoutingDataSource很好的实现了。通过AbstractRoutingDataSource对DataSource的管理，使用apollo配置动态推送能力，动态修改AbstractRoutingDataSource中resolvedDataSources数据源实例，可以很好的实现动态变更线上数据源。

<!--more-->

### 关于AbstractRoutingDataSource

AbstractRoutingDataSource使用Map resolvedDataSources保存了所有可用的数据源实例，预留了一个方法determineCurrentLookupKey给子类去决定使用哪个数据源。这个方法的返回值就是resolvedDataSources中对应的key值，通过这个可以轻松实现应用层面的数据读写分离。当应用程序请求连接时，它拿着子类实现返回的结果去resolvedDataSources中寻找真实的数据源拿数据连接。

### 具体实现

```java
@Configuration
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceConfiguration {

    Logger logger = LoggerFactory.getLogger(getClass());

    private final static String DATASOURCE_TAG = "db";

    @Autowired
    ApplicationContext context;

    @ApolloConfig
    Config config;

    @Bean("dataSource")
    public DynamicDataSource dynamicDataSource() {
        DynamicDataSource source = new DynamicDataSource();
        source.setTargetDataSources(Collections.singletonMap(DATASOURCE_TAG, dataSource()));
        return source;
    }
    @ApolloConfigChangeListener
    public void onChange(ConfigChangeEvent changeEvent) {
        Set changedKeys = changeEvent.changedKeys();
        if (changedKeys.contains("spring.datasource.url")) {
            DynamicDataSource source = context.getBean(DynamicDataSource.class);
            source.setTargetDataSources(Collections.singletonMap(DATASOURCE_TAG, dataSource()));
            source.afterPropertiesSet();
            logger.info("动态切换数据源为：{}", config.getProperty("spring.datasource.url", ""));
        }
    }
    public DataSource dataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(config.getProperty("spring.datasource.url", ""));
        dataSource.setUsername(config.getProperty("spring.datasource.username", ""));
        dataSource.setPassword(config.getProperty("spring.datasource.password", ""));
        return dataSource;
    }
    class DynamicDataSource extends AbstractRoutingDataSource {
        @Override
        protected Object determineCurrentLookupKey() { return DATASOURCE_TAG; }
    }
}
```

### 实例环境说明

1.本文实现基于spring boot2.0的环境，spring boot2.0中默认数据库连接池用的Hikari，这个连接池性能不俗，按官方说法，他们在程序基础数据结构，字节码，编译器级别做了大量优化，来保证Hikari的优异性能。从spirng boot 切换tomcat jdbc默认实现这波操作来看，Hikari这款连接池的性能应该不赖。

2.spring boot默认开启了DataSource的自动装配，可以通过@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)这种方式排除自动装配

### 总结

通过apollo配置动态推送能力，结合AbstractRoutingDataSource，可以轻松实现线上数据源的动态切换。当然，不仅仅是数据库的数据连接可以动态切换，按照上面的设计实现思路，通过apollo的动态配置能力，可以轻松实现很多的线上动态切换。关于线上数据源切换的应用场景，以及apollo的更多应用场景，欢迎大家在下面留言区留言补充。文中提到的Apollo是携程开源的配置中心项目，项目地址如下：https://github.com/ctripcorp/apollo

上文所述实现方式过于简单漏掉了一个重要的问题，老的连接没有做任何的处理，可能造成连接泄漏。其实在我们完全切换成新的数据库连接前，我们需要获取到老的连接池，并且校验是否有活动链接，直到没有任何活动链接时，我们需要关闭老的连接

```java
 private boolean terminateHikariDataSource(HikariDataSource dataSource) {
    HikariPoolMXBean poolMXBean = dataSource.getHikariPoolMXBean();

    //evict idle connections
    poolMXBean.softEvictConnections();

    if (poolMXBean.getActiveConnections() > 0 && retryTimes < MAX_RETRY_TIMES) {
      logger.warn("Data source {} still has {} active connections, will retry in {} ms.", dataSource,
          poolMXBean.getActiveConnections(), RETRY_DELAY_IN_MILLISECONDS);
      return false;
    }

    if (poolMXBean.getActiveConnections() > 0) {
      logger.warn("Retry times({}) >= {}, force closing data source {}, with {} active connections!", retryTimes,
          MAX_RETRY_TIMES, dataSource, poolMXBean.getActiveConnections());
    }

    dataSource.close();

    return true;
  }
```

完整的demo地址：https://github.com/ciweigg2/apollo-dynamic-datasource