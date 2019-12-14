cover: http://ciwei2.cn-sh2.ufileos.com/30.jpg
title: ElasticHD监控elasticsearch数据
date: 2018-10-30 20:33:44
tags: [elasticsearch,elastichd]
categories: [可视化工具]
---
### 一款漂亮的elasticsearch可视化工具

<!--more-->

> github:https://github.com/360EntSecGroup-Skylar/ElasticHD

### docker安装：

```java
docker run -p 9200:9200 -d --name elasticsearch elasticsearch
docker run -p 9800:9800 -d --link elasticsearch:demo containerize/elastichd
Open http://localhost:9800 in Browser
Connect with http://demo:9200
```

![](/images/Elastic HD Dashboard.png)