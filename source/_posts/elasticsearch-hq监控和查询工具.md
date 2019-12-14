cover: http://ciwei2.cn-sh2.ufileos.com/32.jpg
title: elasticsearch-hq监控和查询工具
date: 2018-12-22 14:14:26
tags: [elasticsearch]
categories: [可视化工具]
---
dockerhub地址：https://hub.docker.com/r/elastichq/elasticsearch-hq/

github地址：https://github.com/ElasticHQ/elasticsearch-HQ

<!--more-->

###主要特点
* 适用于2.x，5.x，6.x和当前版本的Elasticsearch。
* 一次监控多个集群。
* 监控节点，索引，碎片和一般群集指标。
* 创建和维护Elasticsearch指数。
* 一键访问ES API和cat API端点。
* 易于使用的查询功能。
* 复制映射并重新索引索引。
* 实时监控重要指标的图表。
* 诊断检查有助于警告具有问题的特定节点。
* 世界各地财富100强企业使用的活跃项目。
* 免费和（真实）开源。;-)

### 安装
```java
docker run -p 5000:5000 elastichq/elasticsearch-hq
Access HQ with: http://localhost:5000
```

![](/images/main_dashboard.png)