cover: http://ciwei2.cn-sh2.ufileos.com/31.jpg
title: elasticsearch sql查询工具
date: 2018-12-21 17:56:25
tags: [elasticsearch]
categories: [综合]
---
### elasticsearch 类似sql的查询工具

源码地址：https://github.com/NLPchina/elasticsearch-sql

<!--more-->

### 安装sql查询插件
```java
./bin/elasticsearch-plugin install https://github.com/NLPchina/elasticsearch-sql/releases/download/6.5.3.0/elasticsearch-sql-6.5.3.0.zip
```

### 安装界面
```java
wget https://github.com/NLPchina/elasticsearch-sql/releases/download/5.4.1.0/es-sql-site-standalone.zip
unzip es-sql-site-standalone.zip
cd site-server
npm install express --save
node node-server.js 
```
### 修改elasticsearch跨域访问
```java
vi elastic6.5.3/elasticsearch-6.5.3/config/elasticsearch.yml
添加：
network.host: 0.0.0.0
http.cors.enabled: true
http.cors.allow-origin: /.*/
```

![](/images/20181221180640.png)