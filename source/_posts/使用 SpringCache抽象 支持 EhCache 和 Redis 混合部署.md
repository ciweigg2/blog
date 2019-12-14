cover: http://ciwei2.cn-sh2.ufileos.com/134.jpg
title: 使用 SpringCache抽象 支持 EhCache 和 Redis 混合部署
date: 2018-08-31 17:43:27
tags: [SpringCache]
categories: [综合]
---
最近项目组在开发本地缓存，其中用到了redis和ehcache，但是在使用注解过程中发现两者会出现冲突，这里给出解决两者冲突的具体方案
<!--more-->
引入redis的依赖：
```java
            <dependency>
                <groupId>redis.clients</groupId>
                <artifactId>jedis</artifactId>
                　　<version>2.9.0</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.data</groupId>
                <artifactId>spring-data-redis</artifactId>
                <version>1.6.0.RELEASE</version>
            </dependency>
```

spring-cache.xml：
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:cache="http://www.springframework.org/schema/cache"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/cache http://www.springframework.org/schema/cache/spring-cache.xsd">

    <bean id="cacheKeyGenerator" class="net.transino.utils.CacheKeyGenerator"/>
    <!-- 启用缓存注解功能 -->
    <cache:annotation-driven cache-manager="cacheManager" key-generator="cacheKeyGenerator"/>
    <!-- Spring提供的基于的Ehcache实现的缓存管理器 -->
    <bean id="cacheManagerFactory" class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean">
        <property name="configLocation" value="classpath:spring/ehcache.xml"/>
    </bean>
    <bean id="cacheManager" class="org.springframework.cache.ehcache.EhCacheCacheManager">
        <property name="cacheManager" ref="cacheManagerFactory"/>
    </bean>

    <!-- redis连接池 -->
    <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxIdle" value="${redis.maxIdle}"/>
        <property name="maxWaitMillis" value="${redis.maxWait}"/>
        <property name="testOnBorrow" value="${redis.testOnBorrow}"/>
    </bean>
    <!-- 连接工厂 -->
    <bean id="JedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
          p:host-name="${redis.host}" p:port="${redis.port}" p:password="${redis.pass}" p:pool-config-ref="poolConfig"/>
    <!-- redis模板 -->
    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="JedisConnectionFactory"/>
    </bean>

    <!-- 配置redisCacheManager -->
    <bean id="redisCacheManager" class="org.springframework.data.redis.cache.RedisCacheManager">
        <constructor-arg name="redisOperations" ref="redisTemplate" />
        <!-- 默认有效期,单位:秒 -->
        <property name="defaultExpiration" value="1800" />
        <!-- 多个缓存有效期,单位:秒 -->
        <property name="expires">
            <map>
                <entry key="ssoSessionFixedTime" value="120" />
                <entry key="fixedTime" value="30" />
                <entry key="sysConfig" value="1800" />
                <entry key="legalPersonList" value="300" />
                <entry key="sysFunctionVo" value="300" />
            </map>
        </property>
    </bean>

</beans>
```

ehcache.xml：
```java
<?xml version="1.0" encoding="UTF-8"?>

<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd" updateCheck="false">
    <!-- 磁盘缓存位置 -->
    <diskStore path="java.io.tmpdir/ehcache"/>
    <defaultCache
            maxElementsInMemory="1000"
            eternal="false"
            timeToIdleSeconds="1800"
            timeToLiveSeconds="1800"
            overflowToDisk="true"/>

    <!--sso-session缓存2分钟固定时间-->
    <cache
            name="ssoSessionFixedTime"
            maxElementsInMemory="500"
            eternal="false"
            timeToLiveSeconds="120"
            overflowToDisk="true"/>

    <!--固定时间缓存配置(到时间后强制清空缓存)  5分钟-->
    <cache
            name="fixedTime"
            maxElementsInMemory="1000"
            eternal="false"
            timeToLiveSeconds="300"
            overflowToDisk="true"/>
    <!--系统配置管理缓存 30分钟 失效-->
    <cache
            name="sysConfig"
            maxElementsInMemory="50"
            eternal="false"
            timeToLiveSeconds="1800"
            overflowToDisk="true"/>
    <!--法人列表缓存 5分钟 失效-->
    <cache
            name="legalPersonList"
            maxElementsInMemory="100"
            eternal="false"
            timeToLiveSeconds="300"
            overflowToDisk="true"/>
    <!--功能列表缓存 5分钟 失效-->
    <cache
            name="sysFunctionVo"
            maxElementsInMemory="1000"
            eternal="false"
            timeToLiveSeconds="300"
            overflowToDisk="true"/>
</ehcache>
```

cache.properties：
```java
redis.host=127.0.0.1
redis.port=6379
redis.pass=
redis.database=0
redis.maxIdle=300
redis.maxWait=3000
redis.testOnBorrow=true

#选择缓存方式 redis ehcache
cache.mode=redis
```

CacheConfig核心配置类：
```java
package net.transino.utils.cache;

import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.Collection;

import lombok.Data;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cache.Cache;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.ehcache.EhCacheCacheManager;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;

@Configuration
@EnableCaching
@Data
public class CacheConfig extends CachingConfigurerSupport {

    @Autowired(required = false)
    private volatile RedisCacheManager redisCacheManager;

    @Autowired(required = false)
    private volatile EhCacheCacheManager ehCacheCacheManager;

    /**
     * 选择缓存
     */
    @Value("${cache.mode}")
    private String cacheMode;

    public CacheConfig() {
        super();
    }

    /**
     * 通过配置实现缓存选择的
     * 该实例是spring操作缓存所必须的，且需要保证唯一，这意味着，如果还需要Spring集成其它缓存中间件，仍然需要集成到这里
     */
    @Bean
    @Override
    public CacheManager cacheManager() {
        return new CacheManager() {
            @Override
            public Collection<String> getCacheNames() {
                Collection<String> cacheNames = new ArrayList<String>();
                if (redisCacheManager != null) {
                    cacheNames.addAll(redisCacheManager.getCacheNames());
                }
                if (ehCacheCacheManager != null) {
                    cacheNames.addAll(ehCacheCacheManager.getCacheNames());
                }
                return cacheNames;
            }

            @Override
            public Cache getCache(String name) {
                if(cacheMode.equals("redis")){
                    return redisCacheManager.getCache(name);
                }else if (cacheMode.equals("ehcache")){
                    return ehCacheCacheManager.getCache(name);
                }else {
                    return null;
                }
            }
        };
    }

    /**
     * 构建默认的键生成器
     */
    @Bean
    @Override
    public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                StringBuilder keyStrBuilder = new StringBuilder();
                keyStrBuilder.append(target.getClass().getName());
                keyStrBuilder.append(method.getName());
                for (Object _param : params) {
                    keyStrBuilder.append(_param.toString());
                }
                return keyStrBuilder.toString();
            }
        };
    }

}
```

这个是基于SpringMvc框架的，接下来就是使用注解了 注解百度有搜索SpringCache注解