title: elasticsearch集群监控
date: 2019-07-29 22:27:42
tags: [elasticsearch]
categories: [综合]
---
### 介绍

es监控感觉是最好的了 支持单机和集群监控 真的很爽呀

<!--more-->

工具地址：https://github.com/lmenezes/cerebro

### 安装

docker安装

```
docker run -p 9000:9000 -d lmenezes/cerebro
```

### 访问

访问界面

http://ip:9000

登录界面输入http://ip:9200

![](/images/20190729123.png)

集群列表界面

![](/images/1149398-20190624140724022-352345765.png)

查看集群状态

![](/images/1149398-20190624145610979-1953235867.png)

修改集群配置

![](/images/1149398-20190624141613295-78055326.png)