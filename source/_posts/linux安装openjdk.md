title: linux安装openjdk
date: 2019-06-24 17:58:49
tags: [linux,openjdk]
categories: [综合]
---
查看yum中管理的可用的JDK软件包列表：

```java
yum search java | grep -i --color JDK
```

<!--more-->

选择合适版本，安装jdk，本人选择的是java-1.8.0-openjdk-devel.x86_64

```java
yum install java-1.8.0-openjdk-devel.x86_64
```

配置环境变量，打开etc文件下profile

```java
vi  /etc/profile
```

在文件内添加

```java
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```

确保这个目录存在有可能会变的/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64

保存关闭后，执行，让配置生效：

```java
source  /etc/profile
```

然后分别输入下面命令确认jdk是否安装成功：

```java
java
```