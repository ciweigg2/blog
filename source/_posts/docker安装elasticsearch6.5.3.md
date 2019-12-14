cover: http://ciwei2.cn-sh2.ufileos.com/14.jpg
title: docker安装elasticsearch6.5.3
date: 2018-12-22 14:06:48
tags: [elasticsearch]
categories: [综合]
---
### docker安装elasticsearch6.5.3并设置跨域和外网访问

<!--more-->

```java
docker run -p 9200:9200 -p 9300:9300 -e "http.cors.enabled=true" -e "http.cors.allow-origin="*"" -d docker.elastic.co/elasticsearch/elasticsearch:6.5.3
```