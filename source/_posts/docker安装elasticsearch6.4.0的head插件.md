cover: http://ciwei2.cn-sh2.ufileos.com/13.jpg
title: docker安装elasticsearch6.4.0的head插件
date: 2018-09-01 21:06:22
tags: [elasticsearch]
categories: [可视化工具]
---
1. elasticsearch6+后的版本head插件是独立出来了，所以需要自己去安装
2. 找了很久才找到这个插件 支持数据查询不报错的
3. 在集群环境下使用head添加索引，需等待界面跳出来({"acknowledged":true,"shards_acknowledged":false,"index":"sss"})，重启集群才会生效
<!--more-->

### 安装：
docker-compose.yml
```
version: '2'
services:
  elasticsearch-head:
      image: wallbase/elasticsearch-head:6-alpine
      container_name: elasticsearch-head
      environment:
        TZ: 'Asia/Shanghai'
      ports:
        - '9100:9100'
```
启动：
docker-compose up -d
访问http://127.0.0.1:9100

![](/images/elasticsearch-head界面.jpg)

之后启动前面文章的elasticsearch6+版本

### 使用：
可以先看一下数据：

![](/images/数据浏览.jpg)

> 查询数据：

![](/images/查询数据.jpg)

> 新增修改数据：

如果id存在则修改数据，如果id不存在则新建数据，如果索引不存在则新建索引存入新的数据呀

需要去掉_search

请求方式改成put

第一个填写索引 第二个填写type 第三个填写id

这边索引和type是一样的 所以填写一样

![](/images/修改数据.jpg)

修改结果：

![](/images/修改结果.jpg)

刚才id为1的数据变成了我们新设置的json值

参考：https://www.cnblogs.com/yanan7890/p/6640289.html