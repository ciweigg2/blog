cover: http://ciwei2.cn-sh2.ufileos.com/3.jpg
title: 搭建Elasticsearch可视化dejavu
date: 2018-09-01 18:22:13
tags: [elasticsearch]
categories: [可视化工具]
---
创建Elasticsearch可视化界面
```
docker run -p 1358:1358 -d appbaseio/dejavu
```
<!--more-->
浏览器访问 http://localhost:1358/live，界面如下 

![](/images/dejavu.jpg)

```
增加索引
curl -X PUT http://localhost:9200/test
```

连接索引

![](/images/dejavu登录的界面.jpg)

效果：

![](/images/QQ截图20180901203130.jpg)