cover: http://ciwei2.cn-sh2.ufileos.com/58.jpg
title: Jenkins官方Docker镜像安装
date: 2019-02-22 17:48:24
tags: [jenkins]
categories: [jenkins]
---
### Jenkins官方Docker镜像安装

<!--more-->

### 下载Jenkins镜像文件

```java
docker search jenkins
docker pull jenkins/jenkins
```

### 启动镜像

```java
mkdir -p /var/jenkins_node
chmod 777 /var/jenkins_node
docker run -d --name myjenkins -p 8080:8080 -p 50000:50000 -v /var/jenkins_node:/var/jenkins_home jenkins/jenkins
```

### 查看logs日志并获取初始密码

```java
docker logs -f myjenkins
```

![](/images/20190124121902119.png)

### 或查看初始化文件，目录就是之前设置的挂载目录

```java
cat /var/jenkins_node/secrets/initialAdminPassword 
```

### 访问

输入Jenkins地址 http://ip:8080/ 输入密码后登录

### 安装插件

maven：Unleash Maven	
ssh：Publish Over SSH