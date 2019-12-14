cover: http://ciwei2.cn-sh2.ufileos.com/148.jpg
title: 安装grafana监控nacos
date: 2019-01-29 17:57:18
tags: [nacos,grafana,prometheus]
categories: [综合]
---
### 下载

```java
git clone https://github.com/nacos-group/nacos-docker.git
cd example
docker-compose -f standalone-mysql.yaml up -d
```

<!--more-->

> 访问 http://ip:3000 用户密码 admin admin

### 配置

![](/images/20190129180155.png)

![](/images/20190129180236.png)

> 导入监控模板

```java
https://github.com/nacos-group/nacos-template/blob/master/nacos-grafana.json
```

参考：https://nacos.io/zh-cn/docs/monitor-guide.html