cover: http://ciwei2.cn-sh2.ufileos.com/91.jpg
title: pinpoint1.8.3本地编译dubbo插件
date: 2019-05-24 16:00:47
tags: [pinpoint,dubbo]
categories: [综合]
---
### dubbo2.7.1的pinpoint1.8.3的插件

pinpoint编译太麻烦了 所以去官网下载编译好的核心类里面有开发插件的核心包的

<!--more-->

https://github.com/naver/pinpoint/releases/download/1.8.3/pinpoint-agent-1.8.3.tar.gz 里面有pinpoint-bootstrap-core-1.8.3.jar和pinpoint-commons-1.8.3.jar

因为官网的插件是老版本的dubbo不支持新版本 所以修改了包名就能用了

### 安装核心类到本地 在jars目录

mvn install:install-file -Dfile=pinpoint-bootstrap-core-1.8.3.jar -DgroupId=com.navercorp.pinpoint -DartifactId=pinpoint-bootstrap-core -Dversion=1.8.3 -Dpackaging=jar

mvn install:install-file -Dfile=pinpoint-commons-1.8.3.jar -DgroupId=com.navercorp.pinpoint -DartifactId=pinpoint-commons -Dversion=1.8.3 -Dpackaging=jar

### 部署一下呀

替换D:\Asd\pinpoint-agent-1.8.3\plugin中的pinpoint-dubbo-plugin-1.8.3.jar

插件地址：https://github.com/ciweigg2/pinpoint-dubbo-plugin

![](/images/20190524155128.png)

![](/images/20190524160607.png)