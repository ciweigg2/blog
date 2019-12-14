title: linux安装maven
date: 2019-08-10 11:36:57
tags: [maven]
categories: [综合]
---
### 软件下载

```
wget  http://mirror.bit.edu.cn/apache/maven/maven-3/3.6.1/binaries/apache-maven-3.6.1-bin.tar.gz
```

<!--more-->

### 安装

```
tar vxf apache-maven-3.6.1-bin.tar.gz
mv apache-maven-3.6.1 /usr/local/maven3
```

### 修改环境变量

```
vi /etc/profile
```

在/etc/profile中添加以下几行

```
MAVEN_HOME=/usr/local/maven3
export MAVEN_HOME
export PATH=${PATH}:${MAVEN_HOME}/bin
```

使配置文件生效

```
source /etc/profile
```

查看是否安装成功

```
mvn -version
```

### 配置maven的仓库地址

配置阿里镜像仓库

```
vi /usr/local/maven3/conf/settings.xml
```

定位到mirrors节点下添加下面配置

```
<!--阿里云镜像仓库 -->
<mirror>
   <id>nexus-aliyun</id>
   <mirrorOf>central</mirrorOf>
   <name>Nexus aliyun</name>
   <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

### 配置本地仓库

定位到这个节点进行编写

```
<localRepository>/home/maven/repo</localRepository>
```