cover: http://ciwei2.cn-sh2.ufileos.com/124.jpg
title: SSM多数据源动态切换
date: 2018-08-26 15:00:42
tags: [多数据源]
categories: [综合]
---
Spring SpringMVC Mybatis动态数据源使用
本人主要有三种方式切换
1、使用aop方式切入
2、使用扫描service层自动切入
3、使用手动设置控制使用数据源
<!--more-->
 
添加aop注解
```java
@Target({ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DynamicRoutingDataSource {

	String value() default "dataSource";

}
```

aop实现类
方式1 扫描serviceImpl：
```java
@Aspect
@Component
public class HandlerDataSourceAop {

	private static Logger logger = LoggerFactory.getLogger(HandlerDataSourceAop.class);

	/**
	 * 方式2 实现类切换
	 * 根据实现类指定的路径切换
	 */
	@Pointcut("execution(public * com.wbb.service.impl.*.*(..))")
	public void pointcut(){};

	@Before(value="pointcut()")
	public void beforeOpt(JoinPoint joinPoint){
		String dataSourceName = "dataSource1";
		MultiDataSource.setDataSourceKey(dataSourceName);
		logger.info("切到"+dataSourceName+"数据库");
	}
	@After(value="pointcut()")
	public void afterOpt(){
		MultiDataSource.toDefault();
		logger.info("切回默认数据库");
	}
}
```

方式2 aop注解
```java
@Aspect
@Component
public class HandlerDataSourceAop {

	private static Logger logger = LoggerFactory.getLogger(HandlerDataSourceAop.class);

	/**
	 * 方式1 注解切换
	 * @within匹配类上的注解
	 * @annotation匹配方法上的注解
	 */
	@Pointcut("@within(com.wbb.annotation.DynamicRoutingDataSource)||@annotation(com.wbb.annotation.DynamicRoutingDataSource)")
	public void pointcut(){}

	@Before(value="pointcut()")
	public void beforeOpt(JoinPoint joinPoint){
		//先查找方法上的注解，没有的话再去查找类上的注解
		MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
		Method method = methodSignature.getMethod();
		DynamicRoutingDataSource annotation = method.getAnnotation(DynamicRoutingDataSource.class);
		if(annotation==null){
			annotation = joinPoint.getTarget().getClass().getAnnotation(DynamicRoutingDataSource.class);
			if(annotation==null){
				return ;
			}
		}
		String dataSourceName = annotation.value();
		MultiDataSource.setDataSourceKey(dataSourceName);
		logger.info("切到"+dataSourceName+"数据库");
	}
	@After(value="pointcut()")
	public void afterOpt(){
		MultiDataSource.toDefault();
		logger.info("切回默认数据库");
	}

}
```

jdbc.properties
```java
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/d
jdbc.username=root
jdbc.password=

jdbc.driverClassName1=com.mysql.jdbc.Driver
jdbc.url1=jdbc:mysql://localhost:3306/d1
jdbc.username1=root
jdbc.password1=
```

spring-mybatis.xml配置
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.2.xsd">

	<context:property-placeholder location="classpath:jdbc.properties" />

	<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
		init-method="init" destroy-method="close">
		<property name="driverClassName" value="${jdbc.driverClassName}" />
		<property name="url" value="${jdbc.url}" />
		<property name="username" value="${jdbc.username}" />
		<property name="password" value="${jdbc.password}" />
		<!-- 配置初始化大小、最小、最大 -->
		<property name="initialSize" value="1" />
		<property name="minIdle" value="1" />
		<property name="maxActive" value="10" />
		<!-- 配置获取连接等待超时的时间 -->
		<property name="maxWait" value="10000" />
		<!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
		<property name="timeBetweenEvictionRunsMillis" value="60000" />
		<!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
		<property name="minEvictableIdleTimeMillis" value="300000" />
		<property name="testWhileIdle" value="true" />
		<!-- 这里建议配置为TRUE，防止取到的连接不可用
		testOnBorrow：申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能
		testOnReturn：归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能 -->
		<property name="testOnBorrow" value="true" />
		<property name="testOnReturn" value="false" />
		<!-- 打开PSCache，并且指定每个连接上PSCache的大小 -->
		<property name="poolPreparedStatements" value="true" />
		<property name="maxPoolPreparedStatementPerConnectionSize"
			value="20" />
		<!-- 开启Druid的监控统计功能 -->
		<property name="filters" value="stat"></property>
	</bean>
		<bean id="dataSource1" class="com.alibaba.druid.pool.DruidDataSource"
		init-method="init" destroy-method="close">
		<property name="driverClassName" value="${jdbc.driverClassName1}" />
		<property name="url" value="${jdbc.url1}" />
		<property name="username" value="${jdbc.username1}" />
		<property name="password" value="${jdbc.password1}" />
		<!-- 配置初始化大小、最小、最大 -->
		<property name="initialSize" value="1" />
		<property name="minIdle" value="1" />
		<property name="maxActive" value="10" />
		<!-- 配置获取连接等待超时的时间 -->
		<property name="maxWait" value="10000" />
		<!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
		<property name="timeBetweenEvictionRunsMillis" value="60000" />
		<!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
		<property name="minEvictableIdleTimeMillis" value="300000" />
		<property name="testWhileIdle" value="true" />
		<!-- 这里建议配置为TRUE，防止取到的连接不可用 -->
		<property name="testOnBorrow" value="true" />
		<property name="testOnReturn" value="false" />
		<!-- 打开PSCache，并且指定每个连接上PSCache的大小 -->
		<property name="poolPreparedStatements" value="true" />
		<property name="maxPoolPreparedStatementPerConnectionSize"
			value="20" />
		<!-- 开启Druid的监控统计功能 -->
		<property name="filters" value="stat"></property>
	</bean>
	<bean id="multiDataSource" class="com.wbb.dataSource.MultiDataSource">
		<property name="defaultTargetDataSource" ref="dataSource"></property>
		<property name="targetDataSources">
			<map>
				<entry key="dataSource1" value-ref="dataSource1"></entry>
			</map>
		</property>
	</bean>
	<!-- mybatis -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="multiDataSource"></property>
		<property name="configLocation" value="classpath:mybatis-config.xml"></property>
		<property name="mapperLocations" value="classpath:com/wbb/mapper/*.xml"></property>
	</bean>
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
		<property name="basePackage" value="com.wbb.mapper"></property>
	</bean>

	<!-- 添加事务管理 -->
	<tx:annotation-driven transaction-manager="transactionManager" />

	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="multiDataSource"></property>
	</bean>

</beans>
```

如何使用：
aop注解的话在方法上或者类上添加注解就行@DynamicRoutingDataSource("dataSource1")
扫描serviceImpl的话 配置好包名就行了
手动添加的话，在实现类方法执行数据库之前添加 MultiDataSource.setDataSourceKey(dataSourceName); 如果需要切回去 MultiDataSource.toDefault();

具体代码参考：[动态数据源](https://github.com/ciweigg2/MultiDataSource)