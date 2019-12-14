cover: http://ciwei2.cn-sh2.ufileos.com/34.jpg
title: elasticsearch安装分词工具ik
date: 2018-09-02 15:37:35
tags: [elasticsearch]
categories: [综合]
---
### 进入容器：
docker exec -it elasticsearch bash
### 版本列表：
https://github.com/medcl/elasticsearch-analysis-ik/releases
### 安装指定版本：
```
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.4.0/elasticsearch-analysis-ik-6.4.0.zip
```
<!--more-->

如果上面报错没有权限 容器中执行
```
chmod -R 755 ./bin/
```

具体效果：

分词有两种模式：ik_smart ，ik_max_word

![](/images/分词.png)