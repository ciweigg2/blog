cover: http://ciwei2.cn-sh2.ufileos.com/80.jpg
title: logback根据大小和时间切分
date: 2018-08-29 11:55:05
tags: [logback]
categories: [综合]
---
设置logback日志的失效天数
设置logback根据大小切分
<!--more-->
logback.xml配置
```java
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
    <!-- 日志输出路径，自定义 -->
    <property name="LOG_HOME" value="C://logs" />

    <!-- 日志文件输出 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${LOG_HOME}/test.log</File>
        <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_HOME}/test_all_%d{yyyy-MM-dd}.%i.log.zip</FileNamePattern>
            <!-- 当天的日志大小 超过${log.max.size}时,压缩日志并保存 -->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志最大保留天数-->
            <maxHistory>1</maxHistory>
        </rollingPolicy>
        <!-- 日志输出的文件的格式  -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%date{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread]%logger{56}.%method\(\):%L -%msg%n</pattern>
        </layout>
    </appender>

    <!-- 日志输出级别 -->
    <root level="INFO">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE" />
    </root>
</configuration>
```