cover: http://ciwei2.cn-sh2.ufileos.com/146.jpg
title: 利用TICK搭建Docker容器可视化监控中心
date: 2018-08-25 18:25:04
tags: [docker性能兼容,tick]
categories: [可视化工具]
---
性能监控是容器服务必不可少的基础设施，容器化应用运行于宿主机上，我们需要知道该容器的运行情况，包括 CPU使用率、内存占用、网络状况以及磁盘空间等等一系列信息。在我的前文《Docker容器可视化监控中心搭建》之中我们就实践过Docker容器的可视化监控，在那篇文章中我们是使用了 cAdvisor + influxdb + grafana 技术栈来完成的。然而容器化世界里向来不会只有一种方法来实现某项功能，可以说有一百条大路来通到罗马，因此本文再来探讨另一种称为 TICK 的技术栈方案来实现Docker容器的性能监控。
<!--more-->
* Telegraf：采用插件机制实现的数据采集服务，可以采集包含Docker容器在内的多种性能数据
* InfluxDB：专门负责存储时序数据
* Chronograf：基于React.js编写的性能数据可视化服务
* Kapacitor：提供告警触发和处理功能
这四个组件组成了性能监控的数据管道：Telegraf负责采集节点上的性能数据，然后放入InfluxDB数据库进行存储，Kapacitor通过监听InfluxDB的性能数据来对异常指标发出告警，而Chronograf用来展示集群实时的各项性能指标和状态，提供一个可视化的界面。

下载官方docker:
```
git clone https://github.com/influxdata/sandbox
cd sandbox
./sandbox up
```

```
所有参数：
$ ./sandbox
sandbox commands:
  up           -> 启动
  down         -> 停止
  restart      -> 重启 sandbox
  influxdb     -> attach to the influx cli
  
  enter (influxdb||kapacitor||chronograf||telegraf||ifql) -> enter the specified container
  logs  (influxdb||kapacitor||chronograf||telegraf||ifql) -> stream logs for the specified container
  
  delete-data  -> 删除TICK Stack创建的所有数据
  docker-clean -> 停止并删除所有正在运行的docker容器
  rebuild-docs -> 重建文档容器以查看更新
```

访问界面：
localhost:8888 - Chronograf's address. You will use this as a management UI for the full stack
localhost:3010 - Documentation server. This contains a simple markdown server for tutorials and documentation.

沙盒启动后，您应该会在浏览器中看到您的信息中心：
![](/images/landing-page.png)

您已准备好开始使用TICK堆栈！
单击左侧导航栏中的“主机”图标以查看主机（已命名telegraf-getting-started）及其总体状态
![](/images/host-list.png)

您可以单击system超链接查看预先构建的仪表板，可视化主机的基本系统统计信息，然后查看教程http://localhost:3010/tutorials。
![](/images/sandbox-dashboard.png)