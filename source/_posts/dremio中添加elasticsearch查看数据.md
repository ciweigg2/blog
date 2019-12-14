cover: http://ciwei2.cn-sh2.ufileos.com/27.jpg
title: dremio中添加elasticsearch查看数据
date: 2018-09-01 18:14:43
tags: [dremio,elasticsearch]
categories: [可视化工具]
---
上一篇文章讲解了怎么安装dremio可视化工具，现在说下怎么添加elasticsearch的查看
<!--more-->

打开dremio，添加Sources

![](/images/创建es来源.jpg)

现在是没有索引的 所以需要添加索引 这个可以通过程序 或者curl -X PUT http://localhost:9200/test添加

添加好了没数据也没关系 程序操作后会有该索引的数据的