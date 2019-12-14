cover: http://ciwei2.cn-sh2.ufileos.com/59.jpg
title: Jenkins部署springboot项目
date: 2019-02-22 22:11:42
tags: [jenkins]
categories: [jenkins]
---
### docker启动的jenkins

挂载的目录是：docker run -d --name myjenkins -p 8080:8080 -p 50000:50000 -v /var/jenkins_node:/var/jenkins_home jenkins/jenkins

<!--more-->

所以我们需要复制maven 和 java到容器挂载的目录/var/jenkins_node

配置自动化全局设置的时候需要使用容器的目录：/var/jenkins_home/jdk /var/jenkins_home/maven

### 安装maven

下载个maven到/var/jenkins_node目录

### 安装jdk

下载个jdk到/var/jenkins_node目录

### 安装git

jenkins自带了git所以不需要安装

### 安装构建后操作插件

Clang Scan-Build

### 安装参数化构建

Git Parameter

### 安装插件

maven：Unleash Maven

ssh：Publish Over SSH

### 配置jenkins全局设置(maven jdk git) 系统管理>全局工具配置

![](/images/20190225213627.png)

![](/images/20190225213646.png)

### 配置Publish Over SSH 系统管理>系统设置

* 免密登录

因为我们是docker安装的 所以需要在docker中配置

在jenkins容器中生成ssh
```java
ssh-keygen -t rsa
ssh-copy-id -i /var/jenkins_home/.ssh/id_rsa.pub root@118.184.218.184
```

* 配置Publish Over SSH

![](/images/20190225214330.png)

### 构建一个新的maven项目

![](/images/20190225214720.png)

![](/images/20190225214759.png)

![](/images/20190225214822.png)

> 参数化构建选择分支

![](/images/20190228174753.png)

### 替换数据库等一些配置(构建前配置)

```java
echo "当前目录在项目根目录"
echo "替换端口为8082"
sed -i "s#^server.port=.*#server.port=8082#g" src/main/resources/application.properties
```

![](/images/20190225222821.png)

> 新建sh脚本 替换stop.sh中的端口就行了

```java
mkdir -p /opt/sh
cd /opt/sh/

vi start.sh

#!/bin/bash
export JAVA_HOME=/var/jenkins_home/jdk1.8
echo ${JAVA_HOME}
export BUILD_ID=dontkillme
echo "授权当前用户"
chmod 777 /opt/dev
echo "执行...."
cd /opt/dev
nohup java -jar *.jar > app.log 2>&1 &
sleep 10s
echo "输出项目启动日志"
cat app.log
echo "启动成功"

vi stop.sh

#!/bin/bash
echo "start stopping the application ..."
PORT=8081
pid=`netstat -anp|grep $PORT|awk '{printf $7}'|cut -d/ -f1`
echo “旧应用进程id：$pid”
if [ -n "$pid" ]
then
kill -9 $pid
fi
```

> Exec command

```java
图片中的执行顺序哟
cd /opt/sh/
chmod -R 777 *.sh
./stop.sh
./start.sh
```

![](/images/20190225214856.png)

> 查看构建的信息

![](/images/20190225215549.png)

> 构建后的war包备份啦

* 备份的目录在宿主机

```java
/var/jenkins_node/jobs/maven-tests/builds/36/archive/target
```

![](/images/20190225215612.png)

参考：

https://www.cnblogs.com/fakerblog/p/8482682.html

https://blog.csdn.net/MenofGod/article/details/81941223

https://blog.csdn.net/weinichendian/article/details/81274065