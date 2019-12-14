cover: http://ciwei2.cn-sh2.ufileos.com/42.jpg
title: Google jib打包docker镜像实战
date: 2019-02-11 16:02:47
tags: [docker,jib]
categories: [综合]
---
### Google jib打包docker镜像实战

参考：https://juejin.im/post/5c60c021f265da2dd37bf85b?utm_source=gold_browser_extension

<!--more-->

```java
    <properties>
        <java.version>1.8</java.version>
        <start-class>com.example.demo.DemoApplication</start-class>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>com.google.cloud.tools</groupId>
                <artifactId>jib-maven-plugin</artifactId>
                <version>1.0.0</version>
                <configuration>
                    <allowInsecureRegistries>true</allowInsecureRegistries>
                    <from>
                        <image>anjia0532/arthas-8-jdk-alpine:latest</image>
                    </from>
                    <to>
                        <image>${project.artifactId}</image>
                        <tags>
                            <tag>latest</tag>
                            <tag>${project.version}</tag>
                        </tags>
                    </to>
                    <container>
                        <mainClass>${start-class}</mainClass>
                        <!--<jvmFlags>-->
                            <!--<jvmFlag>-Xms512m</jvmFlag>-->
                        <!--</jvmFlags>-->
                        <ports>
                            <port>8080</port>
                            <port>5701/udp</port>
                            <port>8563</port>
                        </ports>
                        <entrypoint>
                            <shell>sh</shell>
                            <option>-c</option>
                            <arg>java ${JAVA_OPTS} -cp /app/resources/:/app/classes/:/app/libs/* com.example.demo.DemoApplication
                            </arg>
                        </entrypoint>
                        <appRoot>/app</appRoot>
                        <environment>
                            <SPRING_OUTPUT_ANSI_ENABLED>ALWAYS</SPRING_OUTPUT_ANSI_ENABLED>
                            <JHIPSTER_SLEEP>10</JHIPSTER_SLEEP>
                        </environment>
                        <useCurrentTimestamp>true</useCurrentTimestamp>
                    </container>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

> 构建docker镜像

* 使用jib:dockerBuild是在本地打包镜像，不push到远程，-X参数是显示debug信息
* 如果使用jib:build命令，则打包之后push到远程

```java
mvn clean compile jib:dockerBuild -X
```

* 注意：
pom中用的是 registry.hub.docker.com/hengyunabc/arthas:latest 是 alibaba/arthas (阿里开源的一个Java诊断工具，便于线上调试)封装的docker镜像，如果不需要可以改成 registry.hub.docker.com/openjdk:8-jdk-alpine

> 使用arths：

```java
$ docker run -d --init -p8563:8563 -p8080:8080 -e JAVA_OPTS='-Xms512m -Xmx512m' --name demo demo:latest
## 查看镜像发现所有的参数都加载进来了
$ docker inspect demo
## 下面是启动arthas，如果使用的是openjdk镜像请勿执行
$ docker exec -it demo /bin/sh
$ jid=$(jps | grep App | awk '{print $1}')
$ java -jar /opt/arthas/arthas-boot.jar --target-ip 0.0.0.0 ${jid}
```

* 代码中配置了${JAVA_OPTS} 这样就可以顺利使用JAVA_OPTS环境变量来配置JVM了

> 如果使用了arthas镜像，可以访问 http://ip:8563 ，在页面上填上宿主ip，点击Connect， 然后参考Arthas/命令列表 了解Arthas命令用法