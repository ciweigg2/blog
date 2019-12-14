cover: http://ciwei2.cn-sh2.ufileos.com/46.jpg
title: IDEA使用Cloud Toolkit自动打包发布到服务器
date: 2019-06-30 10:39:44
tags: [CloudToolkit]
categories: [综合]
---
### 介绍

Cloud Toolkit 帮助开发者将本地应用程序一键部署到线下自有 VM，或阿里云 ECS、EDAS 和 Kubernetes 中去
当您每次修改完代码后，是否正在经历反复地打包？采用 SCP 工具上传？使用XShell或SecureCRT登陆服务器？替换部署包？重启？ 
现在开始，请把这些重复繁琐的工作交给 Cloud Toolkit 吧

<!--more-->

### 安装插件

IDEA中搜索Cloud Toolkit安装就行了

### 添加服务器

![](/images/20190630105022.png)

![](/images/20190630105642.png)

### 部署服务(单模块)

![](/images/20190630105511.png)

在linux机器首先创建好启动脚本

```java
mkdir -p /root/act_springboot

cd /root/act_springboot

vi restart.sh

pkill -f test-walle-0.0.1-SNAPSHOT.jar
rm -rf /root/act_springboot/test.log
nohup java -jar /root/act_springboot/test-walle-0.0.1-SNAPSHOT.jar > /root/act_springboot/test.log &

chmod -R 755 restart.sh
```

![](/images/20190630105827.png)

### 部署参数说明


* Deploy File：部署文件包含两种方式

Maven Build：如果当前工程采用 Maven 构建，可以使用 Cloud Toolkit 直接构建并部署

Upload File：如果当前工程并非采用 Maven 构建，或者本地已经存在打包好的部署文件，可以选择并直接上传本地的部署文件

* Target Deploy host：在下拉列表中选择Tag，然后选择要部署的服务器

* Deploy Location ：输入在 ECS 上部署路径，如 /root/tomcat/webapps

* Commond：输入应用启动命令，如 sh /root/restart.sh。表示在完成应用包的部署后，需要执行的命令 —— 对于 Java 程序而言，通常是一句 Tomcat 的启动命令


配置启动后打印启动日志 最后部署完在控制台有个open terminal点击打开就可以看到日志了

![](/images/20190630110134.png)

### 部署服务(多模块)

![](/images/20190630114957.png)

选择想要部署的模块 一般为controller层的呀 但是记住一定要放在父模块构建后面执行哟 不然依赖可能会有问题的

![](/images/20190630114825.png)

所有相关功能需求可以参考：https://help.aliyun.com/document_detail/100310.html?spm=a2c4g.11186623.6.571.3df34909QBXlvY