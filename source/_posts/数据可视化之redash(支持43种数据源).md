title: 数据可视化之redash(支持43种数据源)
date: 2019-07-24 11:14:44
tags: [数据可视化]
categories: [综合]
---
### 安装npm

官网有详细的教程：https://redash.io/

<!--more-->

```
查看当前版本
yum --showduplicates list nodejs | expand
#以10.x 版本为例，要9.x的话只要把该命令中10.x改为9.x就好
curl --silent --location https://rpm.nodesource.com/setup_10.x | bash -
安装10.x版本
yum install nodejs
```

### 安装

```
git clone https://github.com/getredash/redash.git
cd redash/
docker-compose up -d
npm install
创建数据库
docker-compose run --rm server create_db
docker-compose run --rm postgres psql -h postgres -U postgres -c "create database tests"
npm run build
参考：https://redash.io/help/open-source/dev-guide/docker
```

### 配置

添加mysql数据库

![](/images/20190724135133.png)

![](/images/20190724135212.png)

添加Query和Dashboards 需要publish

![](/images/20190724135253.png)

![](/images/20190724135324.png)

生成统计呀 需要publish

![](/images/20190724135347.png)

添加到Dashboards

![](/images/20190724135502.png)

参考：https://juejin.im/post/5d25b88cf265da1bc23f9ff3?utm_source=gold_browser_extension
