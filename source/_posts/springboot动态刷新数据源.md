---
title: springboot动态刷新数据源
author: Ciwei
img: ''
coverImg: ''
top: false
cover: false
toc: true
mathjax: false
password: ''
summary: ''
tags:
  - springboot
  - 动态数据源
categories:
  - 综合
originContent: ''
date: 2019-12-17 16:54:24
---

### 介绍

主要是介绍怎么使用但数据源动态刷新 多数据源原理差不多的 自己研究吧

<!--more-->

废话不多说直接上代码

#### application.yml

默认数据源

```yml
spring:
  # 配置数据源
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/bl2?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=GMT%2b8&allowPublicKeyRetrieval=true
    username: root
    password: 123456
    type: com.alibaba.druid.pool.DruidDataSource
```

#### DataSourceRefresher.java

```java
package com.mxc.springbootmybatisquick.dynamic;

import com.zaxxer.hikari.HikariDataSource;
import com.zaxxer.hikari.HikariPoolMXBean;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.sql.DataSource;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;


@Slf4j
@Component
public class DataSourceRefresher {

    private static final int MAX_RETRY_TIMES = 10;

    @Autowired
    private DynamicDataSource dynamicDataSource;

    private ScheduledExecutorService scheduledExecutorService = Executors.newSingleThreadScheduledExecutor();

    public synchronized void refreshDataSource(DataSource newDataSource) {
        try {
            log.info("refresh data source....");
            DataSource oldDataSource = dynamicDataSource.getAndSetDataSource(newDataSource);
            shutdownDataSourceAsync(oldDataSource);
        } catch (Throwable ex) {
            log.error("refresh data source error", ex);
        }
    }


    private void shutdownDataSourceAsync(DataSource dataSource) {
        scheduledExecutorService.execute(() -> doShutdownDataSource(dataSource));
    }

    /**
     * https://github.com/brettwooldridge/HikariCP/issues/742
     */
    private void doShutdownDataSource(DataSource dataSource) {
        if (dataSource instanceof HikariDataSource) {
            int retryTimes = 0;
            HikariDataSource hikariDataSource = (HikariDataSource) dataSource;
            HikariPoolMXBean poolBean = hikariDataSource.getHikariPoolMXBean();
            while (poolBean.getActiveConnections() > 0 && retryTimes <= MAX_RETRY_TIMES) {
                try {
                    poolBean.softEvictConnections();
                    sleep1Second();
                } catch (Exception e) {
                    log.warn("doShutdownDataSource error ", e);
                } finally {
                    retryTimes++;
                }
            }
            hikariDataSource.close();
            log.info("shutdown data source success");
        }

        // TODO other  DataSource

    }

    private void sleep1Second() {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            //ignore
        }
    }

}
```

#### DynamicDataSource.java

```java
package com.mxc.springbootmybatisquick.dynamic;

import javax.sql.DataSource;
import java.io.PrintWriter;
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.SQLFeatureNotSupportedException;
import java.util.concurrent.atomic.AtomicReference;
import java.util.logging.Logger;

/**
 * @author Ciwei
 * @date 2019/12/2
 */
public final class DynamicDataSource implements DataSource {

    private final AtomicReference<DataSource> dataSourceReference;

    public DynamicDataSource(DataSource dataSource) {
        dataSourceReference = new AtomicReference<>(dataSource);
    }


    DataSource getAndSetDataSource(DataSource newDataSource) {
        return dataSourceReference.getAndSet(newDataSource);
    }

    private DataSource getDataSource() {
        return dataSourceReference.get();
    }

    @Override
    public Connection getConnection() throws SQLException {
        return getDataSource().getConnection();
    }

    @Override
    public Connection getConnection(String username, String password) throws SQLException {
        return getDataSource().getConnection(username, password);
    }

    @Override
    public <T> T unwrap(Class<T> iface) throws SQLException {
        return getDataSource().unwrap(iface);
    }

    @Override
    public boolean isWrapperFor(Class<?> iface) throws SQLException {
        return getDataSource().isWrapperFor(iface);
    }

    @Override
    public PrintWriter getLogWriter() throws SQLException {
        return getDataSource().getLogWriter();
    }

    @Override
    public void setLogWriter(PrintWriter out) throws SQLException {
        getDataSource().setLogWriter(out);
    }

    @Override
    public void setLoginTimeout(int seconds) throws SQLException {
        getDataSource().setLoginTimeout(seconds);
    }

    @Override
    public int getLoginTimeout() throws SQLException {
        return getDataSource().getLoginTimeout();
    }

    @Override
    public Logger getParentLogger() throws SQLFeatureNotSupportedException {
        return getDataSource().getParentLogger();
    }

}
```

#### DynamicDataSourceConfiguration.java

```java
package com.mxc.springbootmybatisquick.dynamic;

import com.zaxxer.hikari.HikariDataSource;
import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author Ciwei
 * @date 2019/12/4
 */
@Configuration
public class DynamicDataSourceConfiguration {

    @Bean
    public DynamicDataSource dynamicDataSource(DataSourceProperties dataSourceProperties) {
        HikariDataSource dataSource = dataSourceProperties.initializeDataSourceBuilder()
                .type(HikariDataSource.class).build();
        return new DynamicDataSource(dataSource);

    }

}
```

#### DynamicRefreshController.java

```java
package com.mxc.springbootmybatisquick.controller;

import com.mxc.springbootmybatisquick.dynamic.DataSourceRefresher;
import com.mxc.springbootmybatisquick.utils.ResponseView;
import com.zaxxer.hikari.HikariDataSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
import org.springframework.web.bind.annotation.*;

/**
 * @author maxiucheng
 * @className DynamicRefreshController
 * @description 刷新数据源
 * @date 2019/12/17 4:00 下午
 **/
@RestController
@RequestMapping(value = "/datasource")
public class DynamicRefreshController {

    @Autowired
    private DataSourceRefresher dataSourceRefresher;

    /**
     * 刷新数据源
     *
     * @param url 数据库连接
     * @param userName 用户名
     * @param password 用户密码
     * @return 返回信息
     */
    @PostMapping(value = "/refresh")
    public ResponseView<String> refresh(String url ,String userName ,String password){
        DataSourceProperties properties = new DataSourceProperties();
        //修改属性
        properties.setUrl(url);
        properties.setUsername(userName);
        properties.setPassword(password);
        HikariDataSource dataSource = properties.initializeDataSourceBuilder()
                .type(HikariDataSource.class).build();
        dataSourceRefresher.refreshDataSource(dataSource);
        return ResponseView.success("刷新成功");
    }

}
```

#### DynamicRefreshProxy.java

```java
package com.mxc.springbootmybatisquick.dynamic;

import org.springframework.util.ReflectionUtils;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.concurrent.atomic.AtomicReference;

/**
 * @author Ciwei
 * @date 2019/12/4
 */
public class DynamicRefreshProxy implements InvocationHandler {

    private final AtomicReference<Object> atomicReference;


    public DynamicRefreshProxy(Object instance) {
        atomicReference = new AtomicReference<>(instance);
    }

    public static Object newInstance(Object obj) {
        return Proxy.newProxyInstance(
                obj.getClass().getClassLoader(),
                obj.getClass().getInterfaces(),
                new DynamicRefreshProxy(obj));
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) {
        return ReflectionUtils.invokeMethod(method, atomicReference.get(), args);
    }

    public static void main(String[] args) {
        //1. 创建 dataSource 代理对象
        //2. 配置刷新之后修改 DynamicRefreshProxy 中的 atomicReference 的引用值
        //3. 修改完之后,关闭关闭旧对象相关的资源
    }

}
```

#### 测试

刷新接口（POST）：

http://localhost:8080/datasource/refresh

请求参数：

`url`：jdbc:mysql://localhost:3306/bl?useUnicode=true%26characterEncoding=utf-8%26useSSL=false%26serverTimezone=GMT%2b8%26allowPublicKeyRetrieval=true

`userName`：root

`password`：123456

请求列表接口（GET）：

http://localhost:8080/api/list

demo：https://github.com/ciweigg2/springboot-mybatis-quick