cover: http://ciwei2.cn-sh2.ufileos.com/43.jpg
title: graylog分布式日志
date: 2019-05-04 09:53:10
tags: [graylog]
categories: [综合]
---
### 安装

* Graylog nodes should have a focus on CPU power. These also serve the user interface to the browser.

* Elasticsearch nodes should have as much RAM as possible and the fastest disks you can get. Everything depends on I/O speed here.

* MongoDB is storing meta information and configuration data and doesn’t need many resources.

<!--more-->

> docker-compose.yml

```java
version: '2'
services:
  # MongoDB: https://hub.docker.com/_/mongo/
  mongodb:
    image: mongo:3
    volumes:
      - mongo_data:/data/db
  # Elasticsearch: https://www.elastic.co/guide/en/elasticsearch/reference/5.6/docker.html
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch-oss:6.6.1
    volumes:
      - es_data:/usr/share/elasticsearch/data
    environment:
      - http.host=0.0.0.0
      - transport.host=localhost
      - network.host=0.0.0.0
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 1g
  # Graylog: https://hub.docker.com/r/graylog/graylog/
  graylog:
    image: graylog/graylog:3.0
    volumes:
      - graylog_journal:/usr/share/graylog/data/journal
    environment:
      # CHANGE ME (must be at least 16 characters)!
      - GRAYLOG_PASSWORD_SECRET=somepasswordpepper
      # Password: admin
      - GRAYLOG_ROOT_PASSWORD_SHA2=8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
      - GRAYLOG_HTTP_EXTERNAL_URI=http://192.168.15.59:9000/
    links:
      - mongodb:mongo
      - elasticsearch
    depends_on:
      - mongodb
      - elasticsearch
    ports:
      # Graylog web interface and REST API
      - 9000:9000
      # Syslog TCP
      - 1514:1514
      # Syslog UDP
      - 1514:1514/udp
      # GELF TCP
      - 12201:12201
      # GELF UDP
      - 12201:12201/udp
# Volumes for persisting data, see https://docs.docker.com/engine/admin/volumes/volumes/
volumes:
  mongo_data:
    driver: local
  es_data:
    driver: local
  graylog_journal:
    driver: local
```

```java
docker-compose up -d
```

### 配置Graylog

在浏览器访问http://ip:9000，如图：

![](/images/graylog0.png)

这里默认用户名密码都是admin，进入后如图所示。

![](/images/graylog1.png)

选择System按钮中的input，录入一个输入源，如图：

![](/images/graylog2.png)

这里以GELF UDP为例，在图中位置选择GELF UDP，选择完成后点击Launch new input，如图

![](/images/graylog3.png)

在Node处选择自己安装的，剩下的就根据需要填写即可，如图：

![](/images/graylog4.png)

保存完成后如图，到这里就已经配置完成了

![](/images/graylog5.png)

### Springboot集成

Logback日志

```java
<dependency>
    <groupId>de.siegmar</groupId>
    <artifactId>logback-gelf</artifactId>
    <version>2.0.0</version>
</dependency>
```

> 项目中集成

```java
	<!-- graylog -->
	<appender name="GELF" class="de.siegmar.logbackgelf.GelfUdpAppender">
		<graylogHost>192.168.15.59</graylogHost>
		<graylogPort>12201</graylogPort>
		<maxChunkSize>508</maxChunkSize>
		<useCompression>true</useCompression>
		<encoder class="de.siegmar.logbackgelf.GelfEncoder">
			<originHost>Ciwei</originHost> <!-- 主机别名 -->
			<includeRawMessage>false</includeRawMessage>
			<includeMarker>true</includeMarker>
			<includeMdcData>true</includeMdcData>
			<includeCallerData>false</includeCallerData>
			<includeRootCauseData>false</includeRootCauseData>
			<includeLevelName>false</includeLevelName>
			<shortPatternLayout class="ch.qos.logback.classic.PatternLayout">
				<pattern>%m%nopex</pattern>
			</shortPatternLayout>
			<fullPatternLayout class="ch.qos.logback.classic.PatternLayout">
				<pattern>%m%n</pattern>
			</fullPatternLayout>
			<staticField>app_name:app-web</staticField> <!-- 添加自定义字段呀 -->
			<staticField>os_arch:${os.arch}</staticField>
			<staticField>os_name:${os.name}</staticField>
			<staticField>os_version:${os.version}</staticField>
		</encoder>
	</appender>

	<logger name="de.codecentric.boot.admin.registry" level="OFF" />
	<root level="info">
		<appender-ref ref="GELF" />
	</root>
```

### 控制台使用

> 搜索

例子：

1.根据source搜索内容为dubbo的

USER-20170729MA是我输出日志机器的名字

```java
source:USER-20170729MA AND message:"DUBBO"
```

2.根据app_name搜索为app-web的app

```java
app_name:app-web
```

![](/images/20190504101351.png)

> 控制台

参考：http://docs.graylog.org/en/3.0/pages/dashboards.html

添加仪表盘：

![](/images/20190504101937.png)

添加内容：

![](/images/20190504101747.png)

显示效果呀：

![](/images/20190504101858.png)

> 官网说明

参考：https://github.com/osiegmar/logback-gelf

> Example

Simple UDP configuration:

```xml
<configuration>

    <appender name="GELF" class="de.siegmar.logbackgelf.GelfUdpAppender">
        <graylogHost>localhost</graylogHost>
        <graylogPort>12201</graylogPort>
    </appender>

    <root level="debug">
        <appender-ref ref="GELF" />
    </root>

</configuration>
```

Simple TCP configuration:

```xml
<configuration>

    <appender name="GELF" class="de.siegmar.logbackgelf.GelfTcpAppender">
        <graylogHost>localhost</graylogHost>
        <graylogPort>12201</graylogPort>
    </appender>

    <root level="debug">
        <appender-ref ref="GELF" />
    </root>

</configuration>
```

Simple TCP with TLS configuration:

```xml
<configuration>

    <appender name="GELF" class="de.siegmar.logbackgelf.GelfTcpTlsAppender">
        <graylogHost>localhost</graylogHost>
        <graylogPort>12201</graylogPort>
    </appender>

    <root level="debug">
        <appender-ref ref="GELF" />
    </root>

</configuration>
```

**Please note, that it is recommended to use Logback's AsyncAppender in conjunction with
GelfTcpAppender or GelfTcpTlsAppender to send logs asynchronously.
See the advanced configuration example below.**


Advanced UDP configuration:

```xml
<configuration>

    <appender name="GELF" class="de.siegmar.logbackgelf.GelfUdpAppender">
        <graylogHost>localhost</graylogHost>
        <graylogPort>12201</graylogPort>
        <maxChunkSize>508</maxChunkSize>
        <useCompression>true</useCompression>
        <encoder class="de.siegmar.logbackgelf.GelfEncoder">
            <originHost>localhost</originHost>
            <includeRawMessage>false</includeRawMessage>
            <includeMarker>true</includeMarker>
            <includeMdcData>true</includeMdcData>
            <includeCallerData>false</includeCallerData>
            <includeRootCauseData>false</includeRootCauseData>
            <includeLevelName>false</includeLevelName>
            <shortPatternLayout class="ch.qos.logback.classic.PatternLayout">
                <pattern>%m%nopex</pattern>
            </shortPatternLayout>
            <fullPatternLayout class="ch.qos.logback.classic.PatternLayout">
                <pattern>%m%n</pattern>
            </fullPatternLayout>
            <staticField>app_name:backend</staticField>
            <staticField>os_arch:${os.arch}</staticField>
            <staticField>os_name:${os.name}</staticField>
            <staticField>os_version:${os.version}</staticField>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="GELF" />
    </root>

</configuration>
```

Advanced TCP configuration:

```xml
<configuration>

    <appender name="GELF" class="de.siegmar.logbackgelf.GelfTcpAppender">
        <graylogHost>localhost</graylogHost>
        <graylogPort>12201</graylogPort>
        <connectTimeout>15000</connectTimeout>
        <reconnectInterval>300</reconnectInterval>
        <maxRetries>2</maxRetries>
        <retryDelay>3000</retryDelay>
        <poolSize>2</poolSize>
        <poolMaxWaitTime>5000</poolMaxWaitTime>
        <encoder class="de.siegmar.logbackgelf.GelfEncoder">
            <originHost>localhost</originHost>
            <includeRawMessage>false</includeRawMessage>
            <includeMarker>true</includeMarker>
            <includeMdcData>true</includeMdcData>
            <includeCallerData>false</includeCallerData>
            <includeRootCauseData>false</includeRootCauseData>
            <includeLevelName>false</includeLevelName>
            <shortPatternLayout class="ch.qos.logback.classic.PatternLayout">
                <pattern>%m%nopex</pattern>
            </shortPatternLayout>
            <fullPatternLayout class="ch.qos.logback.classic.PatternLayout">
                <pattern>%m%n</pattern>
            </fullPatternLayout>
            <staticField>app_name:backend</staticField>
            <staticField>os_arch:${os.arch}</staticField>
            <staticField>os_name:${os.name}</staticField>
            <staticField>os_version:${os.version}</staticField>
        </encoder>
    </appender>

    <appender name="ASYNC GELF" class="ch.qos.logback.classic.AsyncAppender">
        <appender-ref ref="GELF" />
    </appender>

    <root level="debug">
        <appender-ref ref="ASYNC GELF" />
    </root>

</configuration>
```

Advanced TCP with TLS configuration:

```xml
<configuration>

    <appender name="GELF" class="de.siegmar.logbackgelf.GelfTcpTlsAppender">
        <graylogHost>localhost</graylogHost>
        <graylogPort>12201</graylogPort>
        <connectTimeout>15000</connectTimeout>
        <reconnectInterval>300</reconnectInterval>
        <maxRetries>2</maxRetries>
        <retryDelay>3000</retryDelay>
        <poolSize>2</poolSize>
        <poolMaxWaitTime>5000</poolMaxWaitTime>
        <trustAllCertificates>false</trustAllCertificates>
        <encoder class="de.siegmar.logbackgelf.GelfEncoder">
            <originHost>localhost</originHost>
            <includeRawMessage>false</includeRawMessage>
            <includeMarker>true</includeMarker>
            <includeMdcData>true</includeMdcData>
            <includeCallerData>false</includeCallerData>
            <includeRootCauseData>false</includeRootCauseData>
            <includeLevelName>false</includeLevelName>
            <shortPatternLayout class="ch.qos.logback.classic.PatternLayout">
                <pattern>%m%nopex</pattern>
            </shortPatternLayout>
            <fullPatternLayout class="ch.qos.logback.classic.PatternLayout">
                <pattern>%m%n</pattern>
            </fullPatternLayout>
            <staticField>app_name:backend</staticField>
            <staticField>os_arch:${os.arch}</staticField>
            <staticField>os_name:${os.name}</staticField>
            <staticField>os_version:${os.version}</staticField>
        </encoder>
    </appender>

    <appender name="ASYNC GELF" class="ch.qos.logback.classic.AsyncAppender">
        <appender-ref ref="GELF" />
    </appender>

    <root level="debug">
        <appender-ref ref="ASYNC GELF" />
    </root>

</configuration>
```

Configuration
-------------

## Appender

`de.siegmar.logbackgelf.GelfUdpAppender`

* **graylogHost**: IP or hostname of graylog server.
  If the hostname resolves to multiple ip addresses, round robin will be used.
* **graylogPort**: Port of graylog server. Default: 12201.
* **encoder**: See Encoder configuration below.
* **maxChunkSize**: Maximum size of GELF chunks in bytes. Default chunk size is 508 - this prevents
  IP packet fragmentation. This is also the recommended minimum.
  Maximum supported chunk size is 65,467 bytes.
* **useCompression**: If true, compression of GELF messages is enabled. Default: true.


`de.siegmar.logbackgelf.GelfTcpAppender`

* **graylogHost**: IP or hostname of graylog server.
  If the hostname resolves to multiple ip addresses, round robin will be used.
* **graylogPort**: Port of graylog server. Default: 12201.
* **encoder**: See Encoder configuration below.
* **connectTimeout**: Maximum time (in milliseconds) to wait for establishing a connection. A value
  of 0 disables the connect timeout. Default: 15,000 milliseconds.
* **reconnectInterval**: Time interval (in seconds) after an existing connection is closed and
  re-opened. A value of -1 disables automatic reconnects. Default: 60 seconds.
* **maxRetries**: Number of retries. A value of 0 disables retry attempts. Default: 2.
* **retryDelay**: Time (in milliseconds) between retry attempts. Ignored if maxRetries is 0.
  Default: 3,000 milliseconds.
* **poolSize**: Number of concurrent tcp connections (minimum 1). Default: 2.
* **poolMaxWaitTime**: Maximum amount of time (in milliseconds) to wait for a connection to become
  available from the pool. A value of -1 disables the timeout. Default: 5,000 milliseconds.


`de.siegmar.logbackgelf.GelfTcpTlsAppender`

* Everything from GelfTcpAppender
* **trustAllCertificates**: If true, trust all TLS certificates (even self signed certificates).
  You should not use this in production! Default: false.

## Encoder

`de.siegmar.logbackgelf.GelfEncoder`

* **originHost**: Origin hostname - will be auto detected if not specified.
* **includeRawMessage**: If true, the raw message (with argument placeholders) will be sent, too.
  Default: false.
* **includeMarker**: If true, logback markers will be sent, too. Default: true.
* **includeMdcData**: If true, MDC keys/values will be sent, too. Default: true.
* **includeCallerData**: If true, caller data (source file-, method-, class name and line) will be
  sent, too. Default: false.
* **includeRootCauseData**: If true, root cause exception of the exception passed with the log
   message will be exposed in the root_cause_class_name and root_cause_message fields.
   Default: false.
* **includeLevelName**: If true, the log level name (e.g. DEBUG) will be sent, too. Default: false.
* **appendNewline**: If true, a system depended newline separator will be added at the end of each message.
  Don't use this in conjunction with TCP or UDP appenders, as this is only reasonable for
  console logging!
* **shortPatternLayout**: Short message format. Default: `"%m%nopex"`.
* **fullPatternLayout**: Full message format (Stacktrace). Default: `"%m%n"`.
* **staticFields**: Additional, static fields to send to graylog. Defaults: none.
```