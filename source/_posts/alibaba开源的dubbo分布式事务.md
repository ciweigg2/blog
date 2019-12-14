cover: http://ciwei2.cn-sh2.ufileos.com/2.jpg
title: alibaba开源的dubbo分布式事务
date: 2019-01-26 17:17:24
tags: [dubbo,fescar]
categories: [综合]
---
### 分布式事务
FESCAR管理分布式事务的典型生命周期：

* TM要求TC开始新的全球交易。TC生成表示全局事务的XID。
* XID通过微服务的调用链传播。
* RM将本地事务注册为XID到TC的相应全局事务的分支。
* TM要求TC提交或回滚XID的相应全局事务。
* TC在XID的相应全局事务下驱动所有分支事务以完成分支提交或回滚。

<!--more-->

### 本地启动
windows下载https://github.com/alibaba/fescar/releases启动 默认端口8091

### docker方式
fescar-docker:https://github.com/fescar-group/fescar-docker

### 项目集成

```java
            <!--fescar分布式事务-->
            <dependency>
                <groupId>com.alibaba.fescar</groupId>
                <artifactId>fescar-spring</artifactId>
                <version>0.1.2-SNAPSHOT</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba.fescar</groupId>
                <artifactId>fescar-dubbo</artifactId>
                <version>0.1.2-SNAPSHOT</version>
            </dependency>
```

> 添加application.conf

```java
transport {
  type = "TCP"
  server = "NIO"
  thread-factory {
    boss-thread-prefix = "NettyBoss"
    worker-thread-prefix = "NettyServerNIOWorker"
    server-executor-thread-prefix = "NettyServerBizHandler"
    share-boss-worker = false
    client-selector-thread-prefix = "NettyClientSelector"
    client-selector-thread-size = 1
    client-worker-thread-prefix = "NettyClientWorkerThread"
    boss-thread-size = 1
    worker-thread-size = 8
  }
}
service {
  vgroup_mapping.my_test_tx_group = "localRgroup"
  localRgroup.grouplist = "127.0.0.1:8091"
  enableDegrade = false
  disable = false
}
```

> 添加数据源和事务配置

```java
@Configuration
public class AlibabaConfig implements BeanPostProcessor {

	@Autowired
	private Environment environment;

	@Value("${spring.dubbo.application.name}")
	private String dubboApplicaitonId;

	@Value("${dubbo.qos.port}")
	private Integer qosPort;

	@Primary
	@Bean(name = "writeDataSource")
	public DataSource writeDataSource() {
		DruidDataSource dataSource = new DruidDataSource();
		try {
			dataSource.setUrl(this.environment.getProperty("spring.datasource.url"));
			dataSource.setUsername(this.environment.getProperty("spring.datasource.username"));
			dataSource.setPassword(this.environment.getProperty("spring.datasource.password"));
			dataSource.setInitialSize(Integer.parseInt(this.environment.getProperty("spring.datasource.initialSize")));
			dataSource.setMinIdle(Integer.parseInt(this.environment.getProperty("spring.datasource.minIdle")));
			dataSource.setMaxActive(Integer.parseInt(this.environment.getProperty("spring.datasource.maxActive")));
			dataSource.setMaxWait(Long.parseLong(this.environment.getProperty("spring.datasource.maxWait")));
			dataSource.setTimeBetweenEvictionRunsMillis(Integer.parseInt(this.environment.getProperty("spring.datasource.timeBetweenEvictionRunsMillis")));
			dataSource.setMinEvictableIdleTimeMillis(Integer.parseInt(this.environment.getProperty("spring.datasource.minEvictableIdleTimeMillis")));
			dataSource.setValidationQuery(this.environment.getProperty("spring.datasource.validationQuery"));
			dataSource.setTestWhileIdle(Boolean.parseBoolean(this.environment.getProperty("spring.datasource.testWhileIdle")));
			dataSource.setTestOnBorrow(Boolean.parseBoolean(this.environment.getProperty("spring.datasource.testOnBorrow")));
			dataSource.setTestOnReturn(Boolean.parseBoolean(this.environment.getProperty("spring.datasource.testOnReturn")));
			dataSource.setConnectionProperties(this.environment.getProperty("spring.datasource.connectionProperties"));
			dataSource.setFilters(this.environment.getProperty("spring.datasource.filters"));
			dataSource.setDefaultAutoCommit(Boolean.parseBoolean(this.environment.getProperty("spring.datasource.defaultAutoCommit")));
		} catch (SQLException e) {
			e.printStackTrace();
		}
		return dataSource;
	}

	@Bean(name = "accountDataSourceProxy")
	public DataSourceProxy initProxy(@Qualifier("writeDataSource") DataSource druidDatasource) {
		return new DataSourceProxy((DruidDataSource) druidDatasource);
	}

	@Value("${mybatis.mapper-locations}")
	private String mapperLocations;

	@Value("${mybatis.type-aliases-package}")
	private String typeAliasePackage;

	@Bean
	public SqlSessionFactory sqlSessionFactory(DataSourceProxy dataSourceProxy) throws Exception {
		SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
		sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(mapperLocations));
		sqlSessionFactoryBean.setTypeAliasesPackage(typeAliasePackage);
		sqlSessionFactoryBean.setDataSource(dataSourceProxy);
		return sqlSessionFactoryBean.getObject();
	}

	@Bean
	public GlobalTransactionScanner initGlobalTransactionScanner() {
		return new GlobalTransactionScanner(dubboApplicaitonId, "my_test_tx_group");
	}

	@PostConstruct
	public void initFescerClient() {
		System.setProperty("dubbo.qos.port", String.valueOf(qosPort));
		String txServiceGroup = "my_test_tx_group";
		RMClientAT.init(dubboApplicaitonId, txServiceGroup);
	}

	@Bean
	public JdbcTemplate initJdbcTemplate(@Qualifier("accountDataSourceProxy") DataSourceProxy dataSourceProxy) {
		return new JdbcTemplate(dataSourceProxy);
	}

}
```

> 添加application.properties

```java
spring.dubbo.application.name = dubbo-registry-nacos-provider2-sample
dubbo.qos.port=32224
spring.datasource.username=root
spring.datasource.password=
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/fescar?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=UTC
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

mybatis.type-aliases-package=com.mxc.dubbo.model
mybatis.mapper-locations=classpath*:mapper/*Mapper.xml

spring.datasource.initialSize=5
spring.datasource.minIdle=5
spring.datasource.maxActive=100
spring.datasource.maxWait=60000
spring.datasource.timeBetweenEvictionRunsMillis=60000
spring.datasource.minEvictableIdleTimeMillis=300000
spring.datasource.validationQuery=SELECT 1 FROM DUAL
spring.datasource.testWhileIdle=true
spring.datasource.testOnBorrow=false
spring.datasource.testOnReturn=false
spring.datasource.poolPreparedStatements=false
spring.datasource.filters=stat,wall,log4j
spring.datasource.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
spring.datasource.defaultAutoCommit=true
```

> 调用方添加注解

```java
@GlobalTransactional(name = "DemoService.addDemo", timeoutMills = 60000)
```

> 测试模拟异常 查看回滚状态(异常的话分布式项目应该根据特殊异常和自定义抛出的code码去判断，而不是throw new 异常 这样的抛出形式呀)

```java
http://localhost:8080/addDemo
{"success":false,"code":500,"message":"服务器打了个小盹儿~请稍候再试"}
```

具体demo：https://github.com/ciweigg2/springboot-dubbo-nacos-zipkin/tree/dubbo-festcar